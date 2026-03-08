# 🎲 Ordem Paranormal — Guia Completo de Criação do App Mobile
> Stack: **Expo SDK 52 + TypeScript + React Navigation v6 + Zustand + MMKV**

---

## 📁 ESTRUTURA FINAL DE PASTAS

```
ordem-paranormal-app/
├── app/                          ← Expo Router (file-based navigation)
│   ├── _layout.tsx
│   ├── index.tsx                 ← Tela de seleção de personagens
│   ├── create.tsx                ← Criação de personagem (wizard)
│   └── character/
│       ├── _layout.tsx           ← Tab navigator do personagem
│       ├── [id]/
│       │   ├── profile.tsx       ← Perfil (aba 1)
│       │   ├── pericias.tsx      ← Perícias (aba 2)
│       │   ├── inventario.tsx    ← Inventário (aba 3)
│       │   ├── habilidades.tsx   ← Habilidades & Rituais (aba 4)
│       │   └── notas.tsx         ← Notas (aba 5)
├── src/
│   ├── components/
│   │   ├── AttributeWheel.tsx
│   │   ├── StatBox.tsx
│   │   ├── SkillRow.tsx
│   │   ├── InventoryItem.tsx
│   │   ├── AbilityCard.tsx
│   │   ├── ThemeSelector.tsx
│   │   └── ThemedBackground.tsx
│   ├── constants/
│   │   ├── ordemData.ts          ← TODOS os dados do livro
│   │   ├── themes.ts
│   │   └── calculations.ts       ← Cálculos automáticos
│   ├── store/
│   │   ├── characterStore.ts     ← Zustand store
│   │   └── themeStore.ts
│   ├── types/
│   │   └── character.ts          ← Todos os tipos TypeScript
│   └── utils/
│       └── storage.ts            ← MMKV wrapper
├── assets/
│   ├── images/
│   │   ├── bg_red.png            ← b1_2.png renomeado
│   │   ├── bg_blue.png           ← b1_3.png renomeado
│   │   ├── bg_purple.png         ← b1_4.png renomeado
│   │   ├── pentagrama.png        ← Group_1.png (atributos)
│   │   ├── sigil_red.png         ← Group_4.png
│   │   ├── sigil_blue.png        ← Group_7.png
│   │   ├── sigil_purple.png      ← Group_10.png
│   │   └── logo_op.png           ← Group_13.png
│   └── fonts/
├── app.json
├── babel.config.js
├── tsconfig.json
└── package.json
```

---

## PASSO 1 — CRIAR O PROJETO

```bash
# 1. Criar projeto com Expo + TypeScript
npx create-expo-app ordem-paranormal-app --template expo-template-blank-typescript
cd ordem-paranormal-app

# 2. Instalar dependências principais
npx expo install expo-router expo-constants expo-status-bar
npx expo install react-native-safe-area-context react-native-screens
npx expo install react-native-gesture-handler react-native-reanimated

# 3. Navegação e estado
npm install zustand
npx expo install @react-native-async-storage/async-storage

# 4. UI e efeitos visuais
npx expo install expo-linear-gradient
npx expo install expo-blur
npm install react-native-svg
npx expo install expo-image

# 5. Fontes
npx expo install expo-font @expo-google-fonts/cinzel @expo-google-fonts/oswald

# 6. Formulários e validação
npm install react-hook-form zod @hookform/resolvers

# 7. Ícones
npm install @expo/vector-icons

# 8. Storage persistente rápido (substitui AsyncStorage)
npx expo install react-native-mmkv
```

---

## PASSO 2 — CONFIGURAR `app.json`

```json
// app.json
{
  "expo": {
    "name": "Ordem Paranormal",
    "slug": "ordem-paranormal-app",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/images/logo_op.png",
    "userInterfaceStyle": "dark",
    "splash": {
      "image": "./assets/images/bg_red.png",
      "resizeMode": "cover",
      "backgroundColor": "#0a0a0a"
    },
    "ios": {
      "supportsTablet": false,
      "bundleIdentifier": "com.yourname.ordemparanormal"
    },
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/images/logo_op.png",
        "backgroundColor": "#0a0a0a"
      },
      "package": "com.yourname.ordemparanormal"
    },
    "web": {
      "bundler": "metro",
      "output": "static"
    },
    "plugins": [
      "expo-router",
      "expo-font",
      [
        "react-native-mmkv",
        { "ios": { "enableEncryption": false } }
      ]
    ],
    "scheme": "ordemparanormal",
    "experiments": {
      "typedRoutes": true
    }
  }
}
```

---

## PASSO 3 — CONFIGURAR `babel.config.js` e `tsconfig.json`

```js
// babel.config.js
module.exports = function (api) {
  api.cache(true);
  return {
    presets: ['babel-preset-expo'],
    plugins: [
      'react-native-reanimated/plugin',
    ],
  };
};
```

```json
// tsconfig.json
{
  "extends": "expo/tsconfig.base",
  "compilerOptions": {
    "strict": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@assets/*": ["assets/*"]
    }
  }
}
```

---

## PASSO 4 — TIPOS TYPESCRIPT (Base de tudo)

```typescript
// src/types/character.ts

// ─── ATRIBUTOS ───────────────────────────────────────────────
export type AttributeKey = 'FOR' | 'AGI' | 'INT' | 'PRE' | 'VIG';

export interface Attributes {
  FOR: number; // Força — base 1, máx 5 (por NEX)
  AGI: number; // Agilidade
  INT: number; // Intelecto
  PRE: number; // Presença
  VIG: number; // Vigor
}

// ─── ORIGEM ──────────────────────────────────────────────────
export type OrigemKey =
  | 'Academica' | 'Amnesica' | 'Circense' | 'Criatura'
  | 'Curandeira' | 'Desgarrada' | 'Estrangeira' | 'Investigativa'
  | 'Milicia' | 'Nobre' | 'Operativa' | 'Periferica'
  | 'Policial' | 'Religiosa' | 'Rural' | 'Suburbana';

export interface Origem {
  key: OrigemKey;
  nome: string;
  atributos: Partial<Attributes>;       // +1 em dois atributos
  pericias: string[];                    // 2 perícias treinadas
  limiteCredito: CreditoKey;
  equipamentos: string[];
  habilidadeEspecial?: string;
  descricao: string;
}

// ─── CLASSE ──────────────────────────────────────────────────
export type ClasseKey = 'Combatente' | 'Especialista' | 'Ocultista';

// ─── TRILHA ──────────────────────────────────────────────────
export type TrilhaKey =
  // Combatente
  | 'Estrategista' | 'Tanque' | 'Agente'
  // Especialista
  | 'Atirador' | 'Assassino' | 'Investigador' | 'Socorrista'
  // Ocultista
  | 'Linhagem' | 'Paranatural' | 'Sincrético'
  | 'SemTrilha';

// ─── NEX ─────────────────────────────────────────────────────
export type NexKey = 5 | 10 | 15 | 20 | 25 | 30 | 35 | 40 | 45 | 50
  | 55 | 60 | 65 | 70 | 75 | 80 | 85 | 90 | 95 | 99;

// ─── PERÍCIA ─────────────────────────────────────────────────
export type PericiaKey =
  | 'Acrobacia' | 'Adestramento' | 'Artes' | 'Atletismo'
  | 'Atualidades' | 'Ciencias' | 'Crime' | 'Diplomacia'
  | 'Enganacao' | 'Fortitude' | 'Furtividade' | 'Iniciativa'
  | 'Intimidacao' | 'Intuicao' | 'Investigacao' | 'Luta'
  | 'Medicina' | 'Ocultismo' | 'Percepcao' | 'Pilotagem'
  | 'Pontaria' | 'Reflexos' | 'Religiao' | 'Sobrevivencia'
  | 'Tatica' | 'Tecnologia' | 'Vontade';

export interface PericiaStatus {
  treinada: boolean;
  bonus: number; // bônus extra manual
}

export type Pericias = Record<PericiaKey, PericiaStatus>;

// ─── CRÉDITO ─────────────────────────────────────────────────
export type CreditoKey = 'Miseravel' | 'Baixo' | 'Moderado' | 'Alto' | 'Altissimo';

// ─── ITEM / INVENTÁRIO ───────────────────────────────────────
export interface InventoryItem {
  id: string;
  nome: string;
  categoria: 0 | 1 | 2 | 3; // 0=Simples, 1=Tático, 2=Restrito, 3=Sigiloso
  espacos: number;
  descricao?: string;
  munição?: { atual: number; max: number };
}

// ─── HABILIDADE / RITUAL ─────────────────────────────────────
export interface Habilidade {
  id: string;
  nome: string;
  custo: number; // PE
  pagina: number;
  descricao?: string;
  isRitual?: boolean;
  elemento?: string; // Fogo, Água, etc (rituais)
  circulo?: 1 | 2 | 3; // Círculo do ritual
}

// ─── PERSONAGEM COMPLETO ─────────────────────────────────────
export interface Character {
  id: string;
  jogador: string;
  nome: string;
  imagemUri?: string;

  // Escolhas de criação
  origem: OrigemKey;
  classe: ClasseKey;
  trilha: TrilhaKey;
  nex: NexKey;

  // Atributos base (sem bônus)
  atributosBase: Attributes;

  // Derivados (calculados automaticamente)
  pv: { total: number; atual: number };
  pe: { total: number; atual: number };
  sanidade: { total: number; atual: number };
  def: number;
  deslocamento: number;

  // Perícias
  pericias: Pericias;

  // Inventário
  inventario: InventoryItem[];
  limiteItens: number;
  cargaMax: number;

  // Habilidades & Rituais
  habilidades: Habilidade[];
  dtRituais: number;

  // Notas
  notas: string[];

  // Metadados
  criadoEm: string;
  atualizadoEm: string;
  tema: ThemeKey;
}

// ─── TEMA ─────────────────────────────────────────────────────
export type ThemeKey = 'red' | 'blue' | 'purple';
```

---

## PASSO 5 — DADOS DO LIVRO (Regras de Ordem Paranormal)

```typescript
// src/constants/ordemData.ts
import { Origem, AttributeKey, PericiaKey, ClasseKey, TrilhaKey, NexKey } from '../types/character';

// ─── ORIGENS ──────────────────────────────────────────────────
// Cada origem dá: +1 em 2 atributos, 2 perícias treinadas,
// limite de crédito inicial e kit de equipamentos
export const ORIGENS: Record<string, Origem> = {
  Academica: {
    key: 'Academica',
    nome: 'Acadêmica',
    atributos: { INT: 1, PRE: 1 },
    pericias: ['Ciencias', 'Investigacao'],
    limiteCredito: 'Moderado',
    equipamentos: ['Notebook', 'Kit de primeiros socorros'],
    descricao: 'Você passou anos estudando em universidades e institutos de pesquisa.',
  },
  Amnesica: {
    key: 'Amnesica',
    nome: 'Amnésica',
    atributos: { AGI: 1, INT: 1 },
    pericias: ['Percepcao', 'Reflexos'],
    limiteCredito: 'Baixo',
    equipamentos: ['Roupas simples', 'Documento falso'],
    descricao: 'Sua memória foi apagada. Você não sabe quem é, mas seu corpo ainda lembra.',
  },
  Circense: {
    key: 'Circense',
    nome: 'Circense',
    atributos: { AGI: 1, PRE: 1 },
    pericias: ['Acrobacia', 'Artes'],
    limiteCredito: 'Baixo',
    equipamentos: ['Roupas coloridas', 'Kit de malabarismo'],
    descricao: 'Você cresceu nos bastidores de um circo, aprendendo acrobacias e performances.',
  },
  Criatura: {
    key: 'Criatura',
    nome: 'Criatura',
    atributos: { FOR: 1, VIG: 1 },
    pericias: ['Fortitude', 'Percepcao'],
    limiteCredito: 'Miseravel',
    equipamentos: [],
    descricao: 'Você não é completamente humano. Algo paranormal corre em suas veias.',
  },
  Curandeira: {
    key: 'Curandeira',
    nome: 'Curandeira',
    atributos: { INT: 1, PRE: 1 },
    pericias: ['Medicina', 'Religiao'],
    limiteCredito: 'Baixo',
    equipamentos: ['Kit de medicina', 'Ervas medicinais'],
    descricao: 'Você tem o dom de curar, seja através da ciência ou de práticas antigas.',
  },
  Desgarrada: {
    key: 'Desgarrada',
    nome: 'Desgarrada',
    atributos: { AGI: 1, VIG: 1 },
    pericias: ['Furtividade', 'Sobrevivencia'],
    limiteCredito: 'Miseravel',
    equipamentos: ['Faca', 'Roupas velhas'],
    descricao: 'Você cresceu nas ruas, aprendendo a sobreviver sem nada.',
  },
  Estrangeira: {
    key: 'Estrangeira',
    nome: 'Estrangeira',
    atributos: { INT: 1, PRE: 1 },
    pericias: ['Atualidades', 'Diplomacia'],
    limiteCredito: 'Moderado',
    equipamentos: ['Passaporte', 'Dinheiro estrangeiro'],
    descricao: 'Você veio de outro país, trazendo uma perspectiva única.',
  },
  Investigativa: {
    key: 'Investigativa',
    nome: 'Investigativa',
    atributos: { INT: 1, PRE: 1 },
    pericias: ['Investigacao', 'Percepcao'],
    limiteCredito: 'Moderado',
    equipamentos: ['Gravador', 'Câmera'],
    descricao: 'Você é jornalista, detetive ou pesquisador que busca a verdade.',
  },
  Milicia: {
    key: 'Milicia',
    nome: 'Milícia',
    atributos: { FOR: 1, VIG: 1 },
    pericias: ['Luta', 'Tatica'],
    limiteCredito: 'Moderado',
    equipamentos: ['Pistola 9mm', 'Colete tático'],
    descricao: 'Você tem treinamento militar ou paramilitar.',
  },
  Nobre: {
    key: 'Nobre',
    nome: 'Nobre',
    atributos: { INT: 1, PRE: 1 },
    pericias: ['Diplomacia', 'Artes'],
    limiteCredito: 'Altissimo',
    equipamentos: ['Terno/vestido de luxo', 'Celular de última geração'],
    descricao: 'Você vem de família rica e influente.',
  },
  Operativa: {
    key: 'Operativa',
    nome: 'Operativa',
    atributos: { AGI: 1, INT: 1 },
    pericias: ['Crime', 'Furtividade'],
    limiteCredito: 'Alto',
    equipamentos: ['Kit de arrombamento', 'Dispositivo de escuta'],
    descricao: 'Você é agente de campo de alguma organização secreta.',
  },
  Periferica: {
    key: 'Periferica',
    nome: 'Periférica',
    atributos: { FOR: 1, PRE: 1 },
    pericias: ['Atletismo', 'Intimidacao'],
    limiteCredito: 'Baixo',
    equipamentos: ['Canivete', 'Celular básico'],
    descricao: 'Você cresceu nas periferias, aprendendo a se virar.',
  },
  Policial: {
    key: 'Policial',
    nome: 'Policial',
    atributos: { FOR: 1, PRE: 1 },
    pericias: ['Investigacao', 'Luta'],
    limiteCredito: 'Moderado',
    equipamentos: ['Pistola padrão', 'Algemas', 'Distintivo'],
    descricao: 'Você é ou foi policial, com acesso a recursos policiais.',
  },
  Religiosa: {
    key: 'Religiosa',
    nome: 'Religiosa',
    atributos: { PRE: 1, VIG: 1 },
    pericias: ['Religiao', 'Vontade'],
    limiteCredito: 'Baixo',
    equipamentos: ['Símbolo sagrado', 'Livro sagrado'],
    descricao: 'Sua fé é seu escudo.',
  },
  Rural: {
    key: 'Rural',
    nome: 'Rural',
    atributos: { FOR: 1, VIG: 1 },
    pericias: ['Adestramento', 'Sobrevivencia'],
    limiteCredito: 'Baixo',
    equipamentos: ['Faca de caça', 'Kit de caça'],
    descricao: 'Você cresceu no campo, longe das cidades.',
  },
  Suburbana: {
    key: 'Suburbana',
    nome: 'Suburbana',
    atributos: { INT: 1, PRE: 1 },
    pericias: ['Tecnologia', 'Atualidades'],
    limiteCredito: 'Baixo',
    equipamentos: ['Smartphone', 'Mochila'],
    descricao: 'Você é um cidadão comum dos subúrbios.',
  },
};

// ─── PERÍCIAS e seus ATRIBUTOS BASE ──────────────────────────
export const PERICIAS_CONFIG: Record<string, {
  nome: string;
  atributo: AttributeKey;
  treinavel: boolean;
  marcadores: string; // '+' = versátil, '*' = treinável c/ restrição
}> = {
  Acrobacia:     { nome: 'Acrobacia',     atributo: 'AGI', treinavel: true,  marcadores: '+' },
  Adestramento:  { nome: 'Adestramento',  atributo: 'PRE', treinavel: true,  marcadores: '' },
  Artes:         { nome: 'Artes',         atributo: 'PRE', treinavel: true,  marcadores: '*' },
  Atletismo:     { nome: 'Atletismo',     atributo: 'FOR', treinavel: true,  marcadores: '' },
  Atualidades:   { nome: 'Atualidades',   atributo: 'INT', treinavel: true,  marcadores: '' },
  Ciencias:      { nome: 'Ciências',      atributo: 'INT', treinavel: true,  marcadores: '*' },
  Crime:         { nome: 'Crime',         atributo: 'AGI', treinavel: true,  marcadores: '*+' },
  Diplomacia:    { nome: 'Diplomacia',    atributo: 'PRE', treinavel: true,  marcadores: '' },
  Enganacao:     { nome: 'Enganação',     atributo: 'PRE', treinavel: true,  marcadores: '' },
  Fortitude:     { nome: 'Fortitude',     atributo: 'VIG', treinavel: true,  marcadores: '' },
  Furtividade:   { nome: 'Furtividade',   atributo: 'AGI', treinavel: true,  marcadores: '+' },
  Iniciativa:    { nome: 'Iniciativa',    atributo: 'AGI', treinavel: false, marcadores: '' },
  Intimidacao:   { nome: 'Intimidação',   atributo: 'PRE', treinavel: true,  marcadores: '' },
  Intuicao:      { nome: 'Intuição',      atributo: 'PRE', treinavel: false, marcadores: '' },
  Investigacao:  { nome: 'Investigação',  atributo: 'INT', treinavel: true,  marcadores: '' },
  Luta:          { nome: 'Luta',          atributo: 'FOR', treinavel: true,  marcadores: '' },
  Medicina:      { nome: 'Medicina',      atributo: 'INT', treinavel: true,  marcadores: '' },
  Ocultismo:     { nome: 'Ocultismo',     atributo: 'INT', treinavel: true,  marcadores: '' },
  Percepcao:     { nome: 'Percepção',     atributo: 'INT', treinavel: false, marcadores: '' },
  Pilotagem:     { nome: 'Pilotagem',     atributo: 'AGI', treinavel: true,  marcadores: '*' },
  Pontaria:      { nome: 'Pontaria',      atributo: 'AGI', treinavel: true,  marcadores: '' },
  Reflexos:      { nome: 'Reflexos',      atributo: 'AGI', treinavel: false, marcadores: '' },
  Religiao:      { nome: 'Religião',      atributo: 'INT', treinavel: true,  marcadores: '*' },
  Sobrevivencia: { nome: 'Sobrevivência', atributo: 'INT', treinavel: true,  marcadores: '' },
  Tatica:        { nome: 'Tática',        atributo: 'INT', treinavel: true,  marcadores: '' },
  Tecnologia:    { nome: 'Tecnologia',    atributo: 'INT', treinavel: true,  marcadores: '' },
  Vontade:       { nome: 'Vontade',       atributo: 'PRE', treinavel: false, marcadores: '' },
};

// ─── CLASSES ─────────────────────────────────────────────────
export const CLASSES_CONFIG: Record<ClasseKey, {
  nome: string;
  descricao: string;
  pvPorNex: number;    // PV ganho por nível de NEX
  pePorNex: number;    // PE ganho por nível de NEX
  periciasBase: number; // Qtd de perícias que pode treinar
  trilhas: TrilhaKey[];
  resistencias: string[];
}> = {
  Combatente: {
    nome: 'Combatente',
    descricao: 'Especialista em combate físico e uso de armas.',
    pvPorNex: 4,
    pePorNex: 3,
    periciasBase: 3,
    trilhas: ['Estrategista', 'Tanque', 'Agente', 'SemTrilha'],
    resistencias: ['Fortitude', 'Reflexos'],
  },
  Especialista: {
    nome: 'Especialista',
    descricao: 'Mestre em habilidades e perícias variadas.',
    pvPorNex: 3,
    pePorNex: 4,
    periciasBase: 5,
    trilhas: ['Atirador', 'Assassino', 'Investigador', 'Socorrista', 'SemTrilha'],
    resistencias: ['Reflexos', 'Vontade'],
  },
  Ocultista: {
    nome: 'Ocultista',
    descricao: 'Manipulador dos elementos e energias do Outro Lado.',
    pvPorNex: 2,
    pePorNex: 6,
    periciasBase: 2,
    trilhas: ['Linhagem', 'Paranatural', 'Sincrético', 'SemTrilha'],
    resistencias: ['Vontade', 'Fortitude'],
  },
};

// ─── FÓRMULAS DE NEX ─────────────────────────────────────────
// A cada 5% de NEX o personagem evolui
// Pontos de atributo disponíveis crescem com NEX
export const NEX_ATRIBUTOS: Record<number, number> = {
  5: 5,   // 5 pontos para distribuir (base)
  10: 6,  15: 7,  20: 8,  25: 9,
  30: 10, 35: 11, 40: 12, 45: 13, 50: 14,
  55: 15, 60: 16, 65: 17, 70: 18, 75: 19,
  80: 20, 85: 21, 90: 22, 95: 23, 99: 24,
};

// ─── LIMITE DE CRÉDITO ────────────────────────────────────────
export const LIMITE_CREDITO_LABELS: Record<string, string> = {
  Miseravel: 'Miserável',
  Baixo: 'Baixo',
  Moderado: 'Moderado',
  Alto: 'Alto',
  Altissimo: 'Altíssimo',
};

// Valor de crédito (em dinheiro de jogo, referência)
export const LIMITE_CREDITO_VALORES: Record<string, string> = {
  Miseravel:  'Até R$500/mês',
  Baixo:      'Até R$2.000/mês',
  Moderado:   'Até R$5.000/mês',
  Alto:       'Até R$20.000/mês',
  Altissimo:  'Ilimitado (quase)',
};

// ─── BÔNUS DE TREINAMENTO ─────────────────────────────────────
// Perícia não-treinada: 0 | Treinada: +5 | Veterana: +10 | Expert: +15
export const BONUS_TREINAMENTO = {
  naoTreinada: 0,
  treinada: 5,
  veterana: 10,
  expert: 15,
};

// ─── BONUS POR ATRIBUTO ───────────────────────────────────────
// Atributo 1 = 0, 2 = +1, 3 = +2, 4 = +3, 5 = +4
export function getBonusAtributo(valor: number): number {
  return Math.max(0, valor - 1);
}
```

---

## PASSO 6 — CÁLCULOS AUTOMÁTICOS

```typescript
// src/constants/calculations.ts
import { Character, Attributes, ClasseKey, NexKey } from '../types/character';
import { CLASSES_CONFIG, getBonusAtributo } from './ordemData';

// ─── PV TOTAL ────────────────────────────────────────────────
// PV = (pvPorNex da classe × nível NEX) + (VIG × 2)
export function calcPVTotal(classe: ClasseKey, nex: NexKey, vig: number): number {
  const niveles = nex / 5; // NEX 5% = nível 1, 10% = nível 2, etc.
  return (CLASSES_CONFIG[classe].pvPorNex * niveles) + (vig * 2);
}

// ─── PE TOTAL ────────────────────────────────────────────────
// PE = (pePorNex da classe × nível NEX) + PRE
export function calcPETotal(classe: ClasseKey, nex: NexKey, pre: number): number {
  const niveles = nex / 5;
  return (CLASSES_CONFIG[classe].pePorNex * niveles) + pre;
}

// ─── SANIDADE TOTAL ──────────────────────────────────────────
// Sanidade = PRE × 5
export function calcSanidadeTotal(pre: number): number {
  return pre * 5;
}

// ─── DEFESA (DEF) ────────────────────────────────────────────
// DEF base = 10 + bônus AGI
// + bônus de armadura (vem do inventário)
export function calcDEF(agi: number, bonusArmadura: number = 0): number {
  return 10 + getBonusAtributo(agi) + bonusArmadura;
}

// ─── LIMITE DE ITENS ─────────────────────────────────────────
// Limite de itens = FOR + 2
export function calcLimiteItens(forca: number): number {
  return forca + 2;
}

// ─── CARGA MÁXIMA ────────────────────────────────────────────
// Carga máx = FOR × 5 kg
export function calcCargaMax(forca: number): number {
  return forca * 5;
}

// ─── BONUS PERÍCIA ───────────────────────────────────────────
// Total da perícia = d20 + bônus atributo + bônus treinamento + outros
export function calcBonusPericia(
  atributoValor: number,
  treinada: boolean,
  bonusExtra: number = 0
): number {
  return getBonusAtributo(atributoValor) + (treinada ? 5 : 0) + bonusExtra;
}

// ─── DT DE RITUAIS ───────────────────────────────────────────
// DT Rituais = 10 + metade do NEX (arredondado p/ baixo) + INT
export function calcDTRituais(nex: NexKey, int: number): number {
  return 10 + Math.floor(nex / 10) + getBonusAtributo(int);
}

// ─── RECALCULAR TUDO AUTOMATICAMENTE ────────────────────────
export function recalcularPersonagem(char: Character): Partial<Character> {
  const { atributosBase, classe, nex } = char;

  const pvTotal = calcPVTotal(classe, nex, atributosBase.VIG);
  const peTotal = calcPETotal(classe, nex, atributosBase.PRE);
  const sanTotal = calcSanidadeTotal(atributosBase.PRE);
  const def = calcDEF(atributosBase.AGI);
  const limiteItens = calcLimiteItens(atributosBase.FOR);
  const cargaMax = calcCargaMax(atributosBase.FOR);
  const dtRituais = calcDTRituais(nex, atributosBase.INT);

  return {
    pv: { total: pvTotal, atual: Math.min(char.pv.atual, pvTotal) },
    pe: { total: peTotal, atual: Math.min(char.pe.atual, peTotal) },
    sanidade: { total: sanTotal, atual: Math.min(char.sanidade.atual, sanTotal) },
    def,
    limiteItens,
    cargaMax,
    dtRituais,
  };
}
```

---

## PASSO 7 — SISTEMA DE TEMAS

```typescript
// src/constants/themes.ts
import { ThemeKey } from '../types/character';

export interface Theme {
  key: ThemeKey;
  primary: string;
  primaryDark: string;
  primaryLight: string;
  background: string;
  surface: string;
  text: string;
  textSecondary: string;
  border: string;
  inputBg: string;
  danger: string;
  success: string;
  bgImage: any;
  sigilImage: any;
}

export const THEMES: Record<ThemeKey, Theme> = {
  red: {
    key: 'red',
    primary: '#F50603',
    primaryDark: '#A00000',
    primaryLight: '#FF4444',
    background: '#0a0a0a',
    surface: '#1a0000',
    text: '#FFFFFF',
    textSecondary: '#CCCCCC',
    border: '#F50603',
    inputBg: '#1C1C1C',
    danger: '#FF0000',
    success: '#00FF88',
    bgImage: require('@assets/images/bg_red.png'),
    sigilImage: require('@assets/images/sigil_red.png'),
  },
  blue: {
    key: 'blue',
    primary: '#002CFD',
    primaryDark: '#001AA0',
    primaryLight: '#4466FF',
    background: '#000510',
    surface: '#00051a',
    text: '#FFFFFF',
    textSecondary: '#AABBFF',
    border: '#002CFD',
    inputBg: '#0A0A2A',
    danger: '#FF4444',
    success: '#00FF88',
    bgImage: require('@assets/images/bg_blue.png'),
    sigilImage: require('@assets/images/sigil_blue.png'),
  },
  purple: {
    key: 'purple',
    primary: '#8400FB',
    primaryDark: '#5500AA',
    primaryLight: '#AA44FF',
    background: '#06000a',
    surface: '#10001a',
    text: '#FFFFFF',
    textSecondary: '#CCAAFF',
    border: '#8400FB',
    inputBg: '#12001C',
    danger: '#FF4444',
    success: '#00FF88',
    bgImage: require('@assets/images/bg_purple.png'),
    sigilImage: require('@assets/images/sigil_purple.png'),
  },
};
```

---

## PASSO 8 — STORE ZUSTAND (Estado global)

```typescript
// src/store/characterStore.ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import { MMKV } from 'react-native-mmkv';
import { Character, ThemeKey, AttributeKey, PericiaKey } from '../types/character';
import { recalcularPersonagem } from '../constants/calculations';
import { createDefaultPericias } from '../utils/helpers';

// Storage MMKV (muito mais rápido que AsyncStorage)
const storage = new MMKV({ id: 'ordem-paranormal-store' });

const zustandStorage = {
  getItem: (name: string) => storage.getString(name) ?? null,
  setItem: (name: string, value: string) => storage.set(name, value),
  removeItem: (name: string) => storage.delete(name),
};

interface CharacterStore {
  characters: Character[];
  activeCharacterId: string | null;

  // CRUD de personagens
  addCharacter: (char: Character) => void;
  updateCharacter: (id: string, updates: Partial<Character>) => void;
  deleteCharacter: (id: string) => void;
  setActiveCharacter: (id: string) => void;

  // Getters
  getActiveCharacter: () => Character | undefined;

  // Ações de jogo
  setAtributo: (id: string, atributo: AttributeKey, valor: number) => void;
  setPVAtual: (id: string, valor: number) => void;
  setPEAtual: (id: string, valor: number) => void;
  setSanidadeAtual: (id: string, valor: number) => void;
  togglePericiaTreinada: (id: string, pericia: PericiaKey) => void;
  addInventoryItem: (id: string, item: any) => void;
  removeInventoryItem: (id: string, itemId: string) => void;
  updateItemMunicao: (id: string, itemId: string, atual: number) => void;
  addHabilidade: (id: string, hab: any) => void;
  removeHabilidade: (id: string, habId: string) => void;
  updateNota: (id: string, index: number, texto: string) => void;
  setNex: (id: string, nex: any) => void;
}

export const useCharacterStore = create<CharacterStore>()(
  persist(
    (set, get) => ({
      characters: [],
      activeCharacterId: null,

      addCharacter: (char) =>
        set((state) => ({ characters: [...state.characters, char] })),

      updateCharacter: (id, updates) =>
        set((state) => ({
          characters: state.characters.map((c) =>
            c.id === id
              ? { ...c, ...updates, atualizadoEm: new Date().toISOString() }
              : c
          ),
        })),

      deleteCharacter: (id) =>
        set((state) => ({
          characters: state.characters.filter((c) => c.id !== id),
          activeCharacterId:
            state.activeCharacterId === id ? null : state.activeCharacterId,
        })),

      setActiveCharacter: (id) => set({ activeCharacterId: id }),

      getActiveCharacter: () => {
        const { characters, activeCharacterId } = get();
        return characters.find((c) => c.id === activeCharacterId);
      },

      setAtributo: (id, atributo, valor) => {
        set((state) => ({
          characters: state.characters.map((c) => {
            if (c.id !== id) return c;
            const newAtrs = { ...c.atributosBase, [atributo]: valor };
            const recalc = recalcularPersonagem({ ...c, atributosBase: newAtrs });
            return { ...c, atributosBase: newAtrs, ...recalc };
          }),
        }));
      },

      setPVAtual: (id, valor) =>
        set((state) => ({
          characters: state.characters.map((c) =>
            c.id === id
              ? { ...c, pv: { ...c.pv, atual: Math.max(0, Math.min(valor, c.pv.total)) } }
              : c
          ),
        })),

      setPEAtual: (id, valor) =>
        set((state) => ({
          characters: state.characters.map((c) =>
            c.id === id
              ? { ...c, pe: { ...c.pe, atual: Math.max(0, Math.min(valor, c.pe.total)) } }
              : c
          ),
        })),

      setSanidadeAtual: (id, valor) =>
        set((state) => ({
          characters: state.characters.map((c) =>
            c.id === id
              ? {
                  ...c,
                  sanidade: {
                    ...c.sanidade,
                    atual: Math.max(0, Math.min(valor, c.sanidade.total)),
                  },
                }
              : c
          ),
        })),

      togglePericiaTreinada: (id, pericia) =>
        set((state) => ({
          characters: state.characters.map((c) =>
            c.id === id
              ? {
                  ...c,
                  pericias: {
                    ...c.pericias,
                    [pericia]: {
                      ...c.pericias[pericia],
                      treinada: !c.pericias[pericia].treinada,
                    },
                  },
                }
              : c
          ),
        })),

      addInventoryItem: (id, item) =>
        set((state) => ({
          characters: state.characters.map((c) =>
            c.id === id
              ? { ...c, inventario: [...c.inventario, item] }
              : c
          ),
        })),

      removeInventoryItem: (id, itemId) =>
        set((state) => ({
          characters: state.characters.map((c) =>
            c.id === id
              ? { ...c, inventario: c.inventario.filter((i) => i.id !== itemId) }
              : c
          ),
        })),

      updateItemMunicao: (id, itemId, atual) =>
        set((state) => ({
          characters: state.characters.map((c) =>
            c.id === id
              ? {
                  ...c,
                  inventario: c.inventario.map((i) =>
                    i.id === itemId && i.municao
                      ? { ...i, municao: { ...i.municao, atual } }
                      : i
                  ),
                }
              : c
          ),
        })),

      addHabilidade: (id, hab) =>
        set((state) => ({
          characters: state.characters.map((c) =>
            c.id === id
              ? { ...c, habilidades: [...c.habilidades, hab] }
              : c
          ),
        })),

      removeHabilidade: (id, habId) =>
        set((state) => ({
          characters: state.characters.map((c) =>
            c.id === id
              ? { ...c, habilidades: c.habilidades.filter((h) => h.id !== habId) }
              : c
          ),
        })),

      updateNota: (id, index, texto) =>
        set((state) => ({
          characters: state.characters.map((c) => {
            if (c.id !== id) return c;
            const notas = [...c.notas];
            notas[index] = texto;
            return { ...c, notas };
          }),
        })),

      setNex: (id, nex) =>
        set((state) => ({
          characters: state.characters.map((c) => {
            if (c.id !== id) return c;
            const recalc = recalcularPersonagem({ ...c, nex });
            return { ...c, nex, ...recalc };
          }),
        })),
    }),
    {
      name: 'characters-storage',
      storage: createJSONStorage(() => zustandStorage),
    }
  )
);

// ─── THEME STORE ─────────────────────────────────────────────
// src/store/themeStore.ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';

interface ThemeStore {
  theme: ThemeKey;
  setTheme: (theme: ThemeKey) => void;
}

export const useThemeStore = create<ThemeStore>()(
  persist(
    (set) => ({
      theme: 'red',
      setTheme: (theme) => set({ theme }),
    }),
    {
      name: 'theme-storage',
      storage: createJSONStorage(() => zustandStorage),
    }
  )
);
```

---

## PASSO 9 — LAYOUT RAIZ E NAVEGAÇÃO

```typescript
// app/_layout.tsx
import { useEffect } from 'react';
import { Stack } from 'expo-router';
import { StatusBar } from 'expo-status-bar';
import { useFonts, Cinzel_700Bold } from '@expo-google-fonts/cinzel';
import { Oswald_400Regular, Oswald_700Bold } from '@expo-google-fonts/oswald';
import * as SplashScreen from 'expo-splash-screen';
import { GestureHandlerRootView } from 'react-native-gesture-handler';

SplashScreen.preventAutoHideAsync();

export default function RootLayout() {
  const [fontsLoaded] = useFonts({
    Cinzel_700Bold,
    Oswald_400Regular,
    Oswald_700Bold,
  });

  useEffect(() => {
    if (fontsLoaded) SplashScreen.hideAsync();
  }, [fontsLoaded]);

  if (!fontsLoaded) return null;

  return (
    <GestureHandlerRootView style={{ flex: 1 }}>
      <StatusBar style="light" />
      <Stack
        screenOptions={{
          headerShown: false,
          contentStyle: { backgroundColor: '#0a0a0a' },
          animation: 'fade',
        }}
      />
    </GestureHandlerRootView>
  );
}
```

```typescript
// app/character/[id]/_layout.tsx
import { Tabs, useLocalSearchParams } from 'expo-router';
import { View, Text, StyleSheet } from 'react-native';
import { Ionicons, MaterialCommunityIcons } from '@expo/vector-icons';
import { useThemeStore } from '@/store/themeStore';
import { THEMES } from '@/constants/themes';

export default function CharacterTabLayout() {
  const { id } = useLocalSearchParams<{ id: string }>();
  const { theme } = useThemeStore();
  const T = THEMES[theme];

  return (
    <Tabs
      screenOptions={{
        headerShown: false,
        tabBarStyle: {
          backgroundColor: '#0a0a0a',
          borderTopColor: T.primary,
          borderTopWidth: 1,
          height: 70,
          paddingBottom: 10,
        },
        tabBarActiveTintColor: T.primary,
        tabBarInactiveTintColor: '#666',
        tabBarLabelStyle: {
          fontSize: 10,
          fontFamily: 'Oswald_400Regular',
        },
      }}
    >
      <Tabs.Screen
        name="profile"
        options={{
          title: 'Perfil',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="person" size={size} color={color} />
          ),
        }}
      />
      <Tabs.Screen
        name="pericias"
        options={{
          title: 'Perícias',
          tabBarIcon: ({ color, size }) => (
            <MaterialCommunityIcons name="format-list-bulleted" size={size} color={color} />
          ),
        }}
      />
      <Tabs.Screen
        name="inventario"
        options={{
          title: 'Inventário',
          tabBarIcon: ({ color, size }) => (
            <MaterialCommunityIcons name="bag-personal" size={size} color={color} />
          ),
          tabBarIconStyle: { marginTop: -12 },
          tabBarItemStyle: {
            borderRadius: 30,
            backgroundColor: T.primary,
            width: 56,
            height: 56,
            marginTop: -16,
          },
        }}
      />
      <Tabs.Screen
        name="habilidades"
        options={{
          title: 'Habilidades',
          tabBarIcon: ({ color, size }) => (
            <MaterialCommunityIcons name="pentagram" size={size} color={color} />
          ),
        }}
      />
      <Tabs.Screen
        name="notas"
        options={{
          title: 'Notas',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="document-text" size={size} color={color} />
          ),
        }}
      />
    </Tabs>
  );
}
```

---

## PASSO 10 — TELA DE CRIAÇÃO DE PERSONAGEM (Wizard)

```typescript
// app/create.tsx
// Wizard de 5 passos seguindo o livro de O.P.:
// 1. Dados básicos (nome, jogador, tema)
// 2. Origem (escolha + visualização de bônus)
// 3. Classe + Trilha
// 4. Distribuição de atributos
// 5. Perícias iniciais

import { useState } from 'react';
import {
  View, Text, StyleSheet, ScrollView,
  TouchableOpacity, TextInput, Alert, ImageBackground,
} from 'react-native';
import { useRouter } from 'expo-router';
import { useThemeStore } from '@/store/themeStore';
import { useCharacterStore } from '@/store/characterStore';
import { THEMES } from '@/constants/themes';
import { ORIGENS, CLASSES_CONFIG, NEX_ATRIBUTOS } from '@/constants/ordemData';
import { recalcularPersonagem } from '@/constants/calculations';
import { Character, OrigemKey, ClasseKey, TrilhaKey, ThemeKey } from '@/types/character';
import { createDefaultPericias } from '@/utils/helpers';
import 'react-native-get-random-values';
import { v4 as uuidv4 } from 'uuid';

type Step = 1 | 2 | 3 | 4 | 5;

export default function CreateCharacterScreen() {
  const router = useRouter();
  const { theme, setTheme } = useThemeStore();
  const { addCharacter } = useCharacterStore();
  const T = THEMES[theme];

  const [step, setStep] = useState<Step>(1);

  // Dados do wizard
  const [jogador, setJogador] = useState('');
  const [nome, setNome] = useState('');
  const [selectedTheme, setSelectedTheme] = useState<ThemeKey>(theme);
  const [selectedOrigem, setSelectedOrigem] = useState<OrigemKey | null>(null);
  const [selectedClasse, setSelectedClasse] = useState<ClasseKey | null>(null);
  const [selectedTrilha, setSelectedTrilha] = useState<TrilhaKey | null>(null);
  const [atributos, setAtributos] = useState({
    FOR: 1, AGI: 1, INT: 1, PRE: 1, VIG: 1,
  });

  const pontosDisponiveis = NEX_ATRIBUTOS[5]; // NEX inicial 5%
  const pontosUsados = Object.values(atributos).reduce((a, b) => a + b, 0);
  const pontosRestantes = pontosDisponiveis - pontosUsados;

  const ATTR_KEYS = ['FOR', 'AGI', 'INT', 'PRE', 'VIG'] as const;

  function handleFinish() {
    if (!selectedOrigem || !selectedClasse || !selectedTrilha) return;

    const origem = ORIGENS[selectedOrigem];

    // Aplicar bônus de origem nos atributos
    const atributosFinais = { ...atributos };
    Object.entries(origem.atributos).forEach(([k, v]) => {
      (atributosFinais as any)[k] += v;
    });

    const baseChar: Character = {
      id: uuidv4(),
      jogador,
      nome,
      origem: selectedOrigem,
      classe: selectedClasse,
      trilha: selectedTrilha,
      nex: 5,
      atributosBase: atributosFinais,
      pv: { total: 0, atual: 0 },
      pe: { total: 0, atual: 0 },
      sanidade: { total: 0, atual: 0 },
      def: 0,
      deslocamento: 9,
      pericias: createDefaultPericias(origem.pericias),
      inventario: origem.equipamentos.map((eq, i) => ({
        id: `${i}`,
        nome: eq,
        categoria: 0,
        espacos: 1,
      })),
      limiteItens: 0,
      cargaMax: 0,
      habilidades: [],
      dtRituais: 0,
      notas: ['', '', '', ''],
      criadoEm: new Date().toISOString(),
      atualizadoEm: new Date().toISOString(),
      tema: selectedTheme,
    };

    // Recalcular tudo
    const recalc = recalcularPersonagem(baseChar);
    const finalChar: Character = {
      ...baseChar,
      ...recalc,
      pv: { total: recalc.pv!.total, atual: recalc.pv!.total },
      pe: { total: recalc.pe!.total, atual: recalc.pe!.total },
      sanidade: { total: recalc.sanidade!.total, atual: recalc.sanidade!.total },
    };

    addCharacter(finalChar);
    setTheme(selectedTheme);
    router.replace(`/character/${finalChar.id}/profile`);
  }

  // RENDER por step
  return (
    <ImageBackground source={T.bgImage} style={styles.bg}>
      <View style={[styles.overlay, { backgroundColor: 'rgba(0,0,0,0.75)' }]}>

        {/* Header */}
        <View style={styles.header}>
          <Text style={[styles.title, { color: T.primary }]}>
            Criar Personagem
          </Text>
          <Text style={[styles.stepIndicator, { color: T.textSecondary }]}>
            Passo {step} de 5
          </Text>
          {/* Progress bar */}
          <View style={styles.progressBar}>
            <View style={[styles.progressFill, {
              backgroundColor: T.primary,
              width: `${(step / 5) * 100}%`,
            }]} />
          </View>
        </View>

        <ScrollView style={styles.content}>

          {/* ── PASSO 1: Dados básicos ── */}
          {step === 1 && (
            <View>
              <Text style={[styles.stepTitle, { color: T.primary }]}>
                Identificação
              </Text>
              <Text style={styles.label}>Nome do Jogador</Text>
              <TextInput
                style={[styles.input, { borderColor: T.border, color: T.text, backgroundColor: T.inputBg }]}
                value={jogador}
                onChangeText={setJogador}
                placeholder="Seu nome"
                placeholderTextColor="#666"
              />
              <Text style={styles.label}>Nome do Personagem</Text>
              <TextInput
                style={[styles.input, { borderColor: T.border, color: T.text, backgroundColor: T.inputBg }]}
                value={nome}
                onChangeText={setNome}
                placeholder="Nome do seu agente"
                placeholderTextColor="#666"
              />
              <Text style={styles.label}>Tema do App</Text>
              <View style={styles.themeRow}>
                {(['red', 'blue', 'purple'] as ThemeKey[]).map((t) => (
                  <TouchableOpacity
                    key={t}
                    onPress={() => setSelectedTheme(t)}
                    style={[styles.themeCircle, {
                      backgroundColor: THEMES[t].primary,
                      borderWidth: selectedTheme === t ? 3 : 0,
                      borderColor: '#FFF',
                    }]}
                  />
                ))}
              </View>
            </View>
          )}

          {/* ── PASSO 2: Origem ── */}
          {step === 2 && (
            <View>
              <Text style={[styles.stepTitle, { color: T.primary }]}>
                Origem
              </Text>
              <Text style={styles.hint}>
                A origem define sua história e concede bônus de atributos, 
                perícias treinadas e limite de crédito inicial.
              </Text>
              {Object.values(ORIGENS).map((o) => (
                <TouchableOpacity
                  key={o.key}
                  onPress={() => setSelectedOrigem(o.key)}
                  style={[styles.card, {
                    borderColor: selectedOrigem === o.key ? T.primary : T.border,
                    backgroundColor: selectedOrigem === o.key ? T.surface : 'transparent',
                    borderWidth: selectedOrigem === o.key ? 2 : 1,
                  }]}
                >
                  <Text style={[styles.cardTitle, { color: T.text }]}>{o.nome}</Text>
                  <Text style={[styles.cardSub, { color: T.textSecondary }]}>
                    Atributos: {Object.entries(o.atributos).map(([k, v]) => `+${v} ${k}`).join(', ')}
                  </Text>
                  <Text style={[styles.cardSub, { color: T.textSecondary }]}>
                    Perícias: {o.pericias.join(', ')} | Crédito: {o.limiteCredito}
                  </Text>
                  <Text style={[styles.cardDesc, { color: '#888' }]}>{o.descricao}</Text>
                </TouchableOpacity>
              ))}
            </View>
          )}

          {/* ── PASSO 3: Classe + Trilha ── */}
          {step === 3 && (
            <View>
              <Text style={[styles.stepTitle, { color: T.primary }]}>
                Classe & Trilha
              </Text>
              {(Object.keys(CLASSES_CONFIG) as ClasseKey[]).map((classeKey) => {
                const cl = CLASSES_CONFIG[classeKey];
                return (
                  <View key={classeKey}>
                    <TouchableOpacity
                      onPress={() => {
                        setSelectedClasse(classeKey);
                        setSelectedTrilha(null);
                      }}
                      style={[styles.card, {
                        borderColor: selectedClasse === classeKey ? T.primary : T.border,
                        backgroundColor: selectedClasse === classeKey ? T.surface : 'transparent',
                        borderWidth: selectedClasse === classeKey ? 2 : 1,
                      }]}
                    >
                      <Text style={[styles.cardTitle, { color: T.text }]}>{cl.nome}</Text>
                      <Text style={[styles.cardDesc, { color: '#888' }]}>{cl.descricao}</Text>
                      <Text style={[styles.cardSub, { color: T.textSecondary }]}>
                        PV/nível: +{cl.pvPorNex} | PE/nível: +{cl.pePorNex} | Perícias: {cl.periciasBase}
                      </Text>
                    </TouchableOpacity>

                    {/* Trilhas da classe selecionada */}
                    {selectedClasse === classeKey && (
                      <View style={styles.trilhasContainer}>
                        <Text style={[styles.label, { color: T.textSecondary }]}>
                          Escolha uma Trilha:
                        </Text>
                        {cl.trilhas.map((t) => (
                          <TouchableOpacity
                            key={t}
                            onPress={() => setSelectedTrilha(t)}
                            style={[styles.trilhaButton, {
                              borderColor: selectedTrilha === t ? T.primary : '#444',
                              backgroundColor: selectedTrilha === t ? T.primary + '33' : 'transparent',
                            }]}
                          >
                            <Text style={{ color: selectedTrilha === t ? T.primary : '#AAA' }}>
                              {t === 'SemTrilha' ? 'Sem Trilha' : t}
                            </Text>
                          </TouchableOpacity>
                        ))}
                      </View>
                    )}
                  </View>
                );
              })}
            </View>
          )}

          {/* ── PASSO 4: Atributos ── */}
          {step === 4 && (
            <View>
              <Text style={[styles.stepTitle, { color: T.primary }]}>
                Atributos
              </Text>
              <Text style={styles.hint}>
                Distribua {pontosDisponiveis} pontos entre seus atributos 
                (mínimo 1, máximo 3 no NEX inicial 5%).
              </Text>
              <Text style={[styles.pontosRestantes, {
                color: pontosRestantes < 0 ? T.danger : T.primary
              }]}>
                Pontos restantes: {pontosRestantes}
              </Text>
              {ATTR_KEYS.map((attr) => (
                <View key={attr} style={styles.attrRow}>
                  <Text style={[styles.attrLabel, { color: T.text }]}>{attr}</Text>
                  <View style={styles.attrControls}>
                    <TouchableOpacity
                      onPress={() => setAtributos(a => ({ ...a, [attr]: Math.max(1, a[attr] - 1) }))}
                      style={[styles.attrBtn, { backgroundColor: T.surface }]}
                    >
                      <Text style={{ color: T.primary, fontSize: 18 }}>−</Text>
                    </TouchableOpacity>
                    <Text style={[styles.attrValue, { color: T.text }]}>{atributos[attr]}</Text>
                    <TouchableOpacity
                      onPress={() => {
                        if (pontosRestantes > 0 && atributos[attr] < 3) {
                          setAtributos(a => ({ ...a, [attr]: a[attr] + 1 }));
                        }
                      }}
                      style={[styles.attrBtn, { backgroundColor: T.surface }]}
                    >
                      <Text style={{ color: T.primary, fontSize: 18 }}>+</Text>
                    </TouchableOpacity>
                  </View>
                </View>
              ))}

              {/* Prévia do bônus de origem */}
              {selectedOrigem && (
                <View style={[styles.origemBonus, { borderColor: T.border }]}>
                  <Text style={[styles.label, { color: T.textSecondary }]}>
                    Bônus da Origem ({selectedOrigem}) será adicionado:
                  </Text>
                  {Object.entries(ORIGENS[selectedOrigem].atributos).map(([k, v]) => (
                    <Text key={k} style={{ color: T.success }}>
                      +{v} {k} (ficará {(atributos as any)[k] + (v ?? 0)})
                    </Text>
                  ))}
                </View>
              )}
            </View>
          )}

          {/* ── PASSO 5: Resumo e confirmação ── */}
          {step === 5 && selectedOrigem && selectedClasse && (
            <View>
              <Text style={[styles.stepTitle, { color: T.primary }]}>
                Resumo
              </Text>
              <View style={[styles.summaryCard, { borderColor: T.border }]}>
                <Text style={[styles.summaryLine, { color: T.text }]}>
                  👤 {nome} ({jogador})
                </Text>
                <Text style={[styles.summaryLine, { color: T.text }]}>
                  🌍 Origem: {ORIGENS[selectedOrigem].nome}
                </Text>
                <Text style={[styles.summaryLine, { color: T.text }]}>
                  ⚔️ Classe: {selectedClasse} {selectedTrilha && selectedTrilha !== 'SemTrilha' ? `— ${selectedTrilha}` : ''}
                </Text>
                <Text style={[styles.summaryLine, { color: T.text }]}>
                  📊 NEX: 5%
                </Text>
                <Text style={[styles.summaryLine, { color: T.text }]}>
                  Atributos (base):
                  {Object.entries(atributos).map(([k, v]) => ` ${k}:${v}`).join(',')}
                </Text>
                <Text style={[styles.summaryLine, { color: T.success }]}>
                  Perícias treinadas (origem): {ORIGENS[selectedOrigem].pericias.join(', ')}
                </Text>
                <Text style={[styles.summaryLine, { color: T.textSecondary }]}>
                  Equipamentos iniciais: {ORIGENS[selectedOrigem].equipamentos.join(', ')}
                </Text>
                <Text style={[styles.summaryLine, { color: T.textSecondary }]}>
                  Limite de Crédito: {ORIGENS[selectedOrigem].limiteCredito}
                </Text>
              </View>
            </View>
          )}

        </ScrollView>

        {/* Botões de navegação */}
        <View style={styles.navButtons}>
          {step > 1 && (
            <TouchableOpacity
              onPress={() => setStep((s) => (s - 1) as Step)}
              style={[styles.navBtn, { borderColor: T.border }]}
            >
              <Text style={{ color: T.text }}>← Voltar</Text>
            </TouchableOpacity>
          )}
          {step < 5 && (
            <TouchableOpacity
              onPress={() => {
                // Validações por step
                if (step === 1 && (!jogador || !nome)) {
                  Alert.alert('Atenção', 'Preencha o nome do jogador e do personagem.');
                  return;
                }
                if (step === 2 && !selectedOrigem) {
                  Alert.alert('Atenção', 'Escolha uma origem.');
                  return;
                }
                if (step === 3 && (!selectedClasse || !selectedTrilha)) {
                  Alert.alert('Atenção', 'Escolha uma classe e trilha.');
                  return;
                }
                if (step === 4 && pontosRestantes !== 0) {
                  Alert.alert('Atenção', `Você tem ${pontosRestantes} ponto(s) não distribuídos.`);
                  return;
                }
                setStep((s) => (s + 1) as Step);
              }}
              style={[styles.navBtn, { backgroundColor: T.primary }]}
            >
              <Text style={{ color: '#FFF', fontWeight: 'bold' }}>Próximo →</Text>
            </TouchableOpacity>
          )}
          {step === 5 && (
            <TouchableOpacity
              onPress={handleFinish}
              style={[styles.navBtn, { backgroundColor: T.primary }]}
            >
              <Text style={{ color: '#FFF', fontWeight: 'bold' }}>
                ✓ Criar Personagem
              </Text>
            </TouchableOpacity>
          )}
        </View>

      </View>
    </ImageBackground>
  );
}

const styles = StyleSheet.create({
  bg: { flex: 1 },
  overlay: { flex: 1 },
  header: { padding: 20, paddingTop: 60 },
  title: { fontSize: 24, fontFamily: 'Cinzel_700Bold', textAlign: 'center' },
  stepIndicator: { textAlign: 'center', marginTop: 4, fontFamily: 'Oswald_400Regular' },
  progressBar: { height: 3, backgroundColor: '#333', marginTop: 10, borderRadius: 2 },
  progressFill: { height: 3, borderRadius: 2 },
  content: { flex: 1, paddingHorizontal: 20 },
  stepTitle: { fontSize: 20, fontFamily: 'Cinzel_700Bold', marginBottom: 12, textAlign: 'center' },
  label: { color: '#CCC', marginBottom: 6, marginTop: 12, fontFamily: 'Oswald_400Regular' },
  hint: { color: '#888', marginBottom: 12, lineHeight: 18 },
  input: { borderWidth: 1, borderRadius: 6, padding: 10, marginBottom: 4, fontSize: 15 },
  themeRow: { flexDirection: 'row', gap: 12, marginTop: 8 },
  themeCircle: { width: 44, height: 44, borderRadius: 22 },
  card: { borderWidth: 1, borderRadius: 8, padding: 12, marginBottom: 10 },
  cardTitle: { fontSize: 16, fontFamily: 'Oswald_700Bold', marginBottom: 2 },
  cardSub: { fontSize: 12, marginTop: 2 },
  cardDesc: { fontSize: 11, marginTop: 4, lineHeight: 16 },
  trilhasContainer: { paddingLeft: 16, marginTop: 4, marginBottom: 8 },
  trilhaButton: { borderWidth: 1, borderRadius: 6, padding: 8, marginBottom: 6 },
  pontosRestantes: { fontSize: 18, textAlign: 'center', marginBottom: 12, fontFamily: 'Oswald_700Bold' },
  attrRow: { flexDirection: 'row', justifyContent: 'space-between', alignItems: 'center', marginBottom: 10 },
  attrLabel: { fontSize: 16, fontFamily: 'Oswald_700Bold', width: 60 },
  attrControls: { flexDirection: 'row', alignItems: 'center', gap: 12 },
  attrBtn: { width: 36, height: 36, borderRadius: 18, justifyContent: 'center', alignItems: 'center' },
  attrValue: { fontSize: 20, fontFamily: 'Oswald_700Bold', width: 32, textAlign: 'center' },
  origemBonus: { borderWidth: 1, borderRadius: 8, padding: 10, marginTop: 16 },
  summaryCard: { borderWidth: 1, borderRadius: 8, padding: 16 },
  summaryLine: { marginBottom: 6, lineHeight: 20 },
  navButtons: { flexDirection: 'row', justifyContent: 'space-between', padding: 20, gap: 12 },
  navBtn: { flex: 1, padding: 14, borderRadius: 8, borderWidth: 1, alignItems: 'center' },
});
```

---

## PASSO 11 — TELA DE PERFIL

```typescript
// app/character/[id]/profile.tsx
import { ScrollView, View, Text, StyleSheet, TouchableOpacity, ImageBackground, Image } from 'react-native';
import { useLocalSearchParams } from 'expo-router';
import { useCharacterStore } from '@/store/characterStore';
import { useThemeStore } from '@/store/themeStore';
import { THEMES } from '@/constants/themes';
import AttributeWheel from '@/components/AttributeWheel';
import StatBox from '@/components/StatBox';

export default function ProfileScreen() {
  const { id } = useLocalSearchParams<{ id: string }>();
  const char = useCharacterStore((s) => s.characters.find((c) => c.id === id));
  const { setPVAtual, setPEAtual, setSanidadeAtual } = useCharacterStore();
  const { theme } = useThemeStore();
  const T = THEMES[char?.tema ?? theme];

  if (!char) return null;

  return (
    <ImageBackground source={T.bgImage} style={{ flex: 1 }}>
      <View style={{ flex: 1, backgroundColor: 'rgba(0,0,0,0.80)' }}>
        <ScrollView contentContainerStyle={styles.container}>

          {/* Header */}
          <Text style={[styles.gameTitle, { color: T.primary }]}>Ordem Paranormal</Text>

          {/* Jogador / Personagem */}
          <View style={styles.infoRow}>
            <Text style={styles.infoLabel}>Jogador:</Text>
            <View style={[styles.infoBox, { borderColor: T.border }]}>
              <Text style={styles.infoText}>{char.jogador}</Text>
            </View>
          </View>
          <Text style={styles.infoLabel}>Personagem:</Text>
          <View style={[styles.infoBox, { borderColor: T.border }]}>
            <Text style={styles.infoText}>{char.nome}</Text>
          </View>

          {/* Avatar + Origem/Classe/Trilha/Patente */}
          <View style={styles.midRow}>
            <View style={[styles.avatarContainer, { borderColor: T.border }]}>
              {/* Aqui você coloca expo-image-picker para foto do personagem */}
              <Text style={{ color: '#555', fontSize: 12, textAlign: 'center' }}>
                Toque para{'\n'}adicionar foto
              </Text>
            </View>
            <View style={styles.rightInfo}>
              <Text style={[styles.rightLabel, { color: T.primary }]}>Origem:</Text>
              <View style={[styles.rightBox, { borderColor: T.border }]}>
                <Text style={styles.rightText}>{char.origem}</Text>
              </View>
              <Text style={[styles.rightLabel, { color: T.primary }]}>Classe:</Text>
              <View style={[styles.rightBox, { borderColor: T.border }]}>
                <Text style={styles.rightText}>{char.classe}</Text>
              </View>
              <Text style={[styles.rightLabel, { color: T.primary }]}>Trilha:</Text>
              <View style={[styles.rightBox, { borderColor: T.border }]}>
                <Text style={styles.rightText}>{char.trilha === 'SemTrilha' ? 'Sem trilha' : char.trilha}</Text>
              </View>
              <Text style={[styles.rightLabel, { color: T.primary }]}>Patente:</Text>
              <View style={[styles.rightBox, { borderColor: T.border }]}>
                <Text style={styles.rightText}>{getNexPatente(char.nex)}</Text>
              </View>
            </View>
          </View>

          {/* Stats rápidos */}
          <View style={styles.statsRow}>
            <StatBox label="DESLOC:" value={`${char.deslocamento}m`} theme={T} />
            <View style={styles.peContainer}>
              <Text style={[styles.statsLabel, { color: T.primary }]}>PE:</Text>
              {/* Botão - e + para PE */}
              <TouchableOpacity onPress={() => setPEAtual(id, char.pe.atual - 1)}>
                <Text style={[styles.bigNum, { color: T.text }]}>{char.pe.atual}</Text>
              </TouchableOpacity>
              <Text style={{ color: T.textSecondary }}>{char.pe.total}</Text>
            </View>
          </View>

          {/* NEX */}
          <View style={styles.nexRow}>
            <Text style={[styles.statsLabel, { color: T.primary }]}>NEX:</Text>
            <View style={[styles.nexBox, { borderColor: T.border }]}>
              <Text style={{ color: T.text }}>{char.nex}%</Text>
            </View>
            <Text style={{ color: T.textSecondary }}>Fé por rodada: {calcFePorRodada(char.nex)}</Text>
          </View>

          {/* DEF */}
          <View style={styles.defRow}>
            <Text style={[styles.defLabel, { color: T.text }]}>DEF:</Text>
            <View style={[styles.defBox, { borderColor: T.border }]}>
              <Text style={[styles.defValue, { color: T.text }]}>{char.def}</Text>
            </View>
          </View>

          {/* Roda de Atributos */}
          <AttributeWheel
            attributes={char.atributosBase}
            theme={T}
            onAttributeChange={(attr, val) => {
              /* useCharacterStore().setAtributo(id, attr, val) */
            }}
          />

          {/* PV */}
          <View style={styles.pvRow}>
            <Text style={[styles.statsLabel, { color: T.primary }]}>PV:</Text>
            <View style={[styles.pvBox, { borderColor: T.border }]}>
              <Text style={{ color: T.text }}>{char.pv.total}</Text>
            </View>
            <Text style={{ color: T.textSecondary }}>Total</Text>
            <View style={[styles.pvBox, { borderColor: T.border }]}>
              <Text style={{ color: T.text }}>{char.pv.atual}</Text>
            </View>
            <Text style={{ color: T.textSecondary }}>Atual</Text>
            {/* + e - */}
            <TouchableOpacity
              onPress={() => setPVAtual(id, char.pv.atual + 1)}
              style={[styles.plusMinusBtn, { backgroundColor: T.primary }]}
            >
              <Text style={{ color: '#FFF', fontSize: 20 }}>+</Text>
            </TouchableOpacity>
            <TouchableOpacity
              onPress={() => setPVAtual(id, char.pv.atual - 1)}
              style={[styles.plusMinusBtn, { backgroundColor: T.surface }]}
            >
              <Text style={{ color: T.primary, fontSize: 20 }}>−</Text>
            </TouchableOpacity>
          </View>

          {/* Sanidade */}
          <View style={styles.sanRow}>
            <Text style={[styles.statsLabel, { color: T.primary }]}>Sanidade:</Text>
            <View style={[styles.pvBox, { borderColor: T.border }]}>
              <Text style={{ color: T.text }}>{char.sanidade.atual}</Text>
            </View>
            <Text style={{ color: T.textSecondary }}>Atual</Text>
            <View style={[styles.pvBox, { borderColor: T.border }]}>
              <Text style={{ color: T.text }}>{char.sanidade.total}</Text>
            </View>
            <Text style={{ color: T.textSecondary }}>Total</Text>
          </View>

        </ScrollView>
      </View>
    </ImageBackground>
  );
}

function getNexPatente(nex: number): string {
  if (nex <= 10) return 'Recruta';
  if (nex <= 20) return 'Operativo';
  if (nex <= 35) return 'Agente';
  if (nex <= 50) return 'Especial';
  if (nex <= 65) return 'Veterano';
  if (nex <= 80) return 'Élite';
  return 'Lendário';
}

function calcFePorRodada(nex: number): number {
  return Math.floor(nex / 10) + 1;
}

const styles = StyleSheet.create({
  container: { padding: 20, paddingTop: 60 },
  gameTitle: { fontSize: 26, fontFamily: 'Cinzel_700Bold', textAlign: 'center', marginBottom: 16 },
  infoRow: { flexDirection: 'row', alignItems: 'center', gap: 8, marginBottom: 6 },
  infoLabel: { color: '#AAA', fontFamily: 'Oswald_400Regular', minWidth: 80 },
  infoBox: { flex: 1, borderWidth: 1, borderRadius: 4, padding: 8 },
  infoText: { color: '#FFF', fontFamily: 'Oswald_400Regular' },
  midRow: { flexDirection: 'row', gap: 12, marginTop: 12 },
  avatarContainer: { width: 140, height: 180, borderWidth: 1, borderRadius: 8, justifyContent: 'center', alignItems: 'center' },
  rightInfo: { flex: 1 },
  rightLabel: { fontSize: 11, textAlign: 'right', fontFamily: 'Oswald_400Regular' },
  rightBox: { borderWidth: 1, borderRadius: 4, padding: 6, marginBottom: 4 },
  rightText: { color: '#FFF', textAlign: 'center', fontFamily: 'Oswald_400Regular' },
  statsRow: { flexDirection: 'row', gap: 12, marginTop: 16 },
  statsLabel: { fontFamily: 'Oswald_700Bold', fontSize: 14 },
  bigNum: { fontSize: 24, fontFamily: 'Oswald_700Bold' },
  peContainer: { flexDirection: 'row', alignItems: 'center', gap: 8 },
  nexRow: { flexDirection: 'row', alignItems: 'center', gap: 8, marginTop: 8 },
  nexBox: { borderWidth: 1, borderRadius: 4, paddingHorizontal: 12, paddingVertical: 4 },
  defRow: { flexDirection: 'row', alignItems: 'center', gap: 12, marginTop: 12 },
  defLabel: { fontSize: 18, fontFamily: 'Oswald_700Bold' },
  defBox: { borderWidth: 1, borderRadius: 4, paddingHorizontal: 16, paddingVertical: 6 },
  defValue: { fontSize: 18, fontFamily: 'Oswald_700Bold' },
  pvRow: { flexDirection: 'row', alignItems: 'center', gap: 8, marginTop: 16 },
  pvBox: { borderWidth: 1, borderRadius: 4, paddingHorizontal: 10, paddingVertical: 4 },
  plusMinusBtn: { width: 40, height: 40, borderRadius: 20, justifyContent: 'center', alignItems: 'center' },
  sanRow: { flexDirection: 'row', alignItems: 'center', gap: 8, marginTop: 8 },
});
```

---

## PASSO 12 — COMPONENTE RODA DE ATRIBUTOS

```typescript
// src/components/AttributeWheel.tsx
// Renderiza o pentagrama com atributos nos vértices
// usando SVG (react-native-svg)
import React from 'react';
import { View, Text, Image, StyleSheet, TouchableOpacity } from 'react-native';
import Svg, { Polygon, Circle, Line, Text as SvgText } from 'react-native-svg';
import { Attributes, AttributeKey } from '@/types/character';
import { Theme } from '@/constants/themes';

interface Props {
  attributes: Attributes;
  theme: Theme;
  onAttributeChange: (attr: AttributeKey, val: number) => void;
}

const ATTR_POSITIONS = {
  AGI: { x: 100, y: 10,  label: 'AGILIDADE' },
  INT: { x: 185, y: 80,  label: 'INTELECTO' },
  VIG: { x: 150, y: 175, label: 'VIGOR' },
  PRE: { x: 50,  y: 175, label: 'PRESENÇA' },
  FOR: { x: 15,  y: 80,  label: 'FORÇA' },
};

export default function AttributeWheel({ attributes, theme: T, onAttributeChange }: Props) {
  return (
    <View style={styles.container}>
      {/* Imagem do pentagrama como fundo */}
      <Image
        source={require('@assets/images/pentagrama.png')}
        style={styles.pentagram}
      />

      {/* Centro */}
      <View style={styles.center}>
        <Text style={[styles.centerText, { color: T.primary }]}>ATRIBUTOS</Text>
      </View>

      {/* Atributo: AGI (topo) */}
      <AttrNode
        attr="AGI"
        label="AGILIDADE"
        value={attributes.AGI}
        style={styles.agiPos}
        theme={T}
        onChange={onAttributeChange}
      />
      {/* Atributo: INT (direita) */}
      <AttrNode
        attr="INT"
        label="INTELECTO"
        value={attributes.INT}
        style={styles.intPos}
        theme={T}
        onChange={onAttributeChange}
      />
      {/* Atributo: VIG (baixo-direita) */}
      <AttrNode
        attr="VIG"
        label="VIGOR"
        value={attributes.VIG}
        style={styles.vigPos}
        theme={T}
        onChange={onAttributeChange}
      />
      {/* Atributo: PRE (baixo-esquerda) */}
      <AttrNode
        attr="PRE"
        label="PRESENÇA"
        value={attributes.PRE}
        style={styles.prePos}
        theme={T}
        onChange={onAttributeChange}
      />
      {/* Atributo: FOR (esquerda) */}
      <AttrNode
        attr="FOR"
        label="FORÇA"
        value={attributes.FOR}
        style={styles.forPos}
        theme={T}
        onChange={onAttributeChange}
      />
    </View>
  );
}

function AttrNode({
  attr, label, value, style, theme: T, onChange,
}: {
  attr: AttributeKey; label: string; value: number;
  style: any; theme: Theme;
  onChange: (a: AttributeKey, v: number) => void;
}) {
  return (
    <View style={[styles.attrNode, style]}>
      <TouchableOpacity onPress={() => onChange(attr, Math.max(1, value - 1))}>
        <Text style={[styles.nodeValue, { color: T.text }]}>{value}</Text>
      </TouchableOpacity>
      <Text style={[styles.nodeLabel, { color: T.primary }]}>{label}</Text>
      <Text style={[styles.nodeAttr, { color: T.textSecondary }]}>{attr}</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { width: 240, height: 240, alignSelf: 'center', marginTop: 16 },
  pentagram: { position: 'absolute', width: 240, height: 240, opacity: 0.4 },
  center: {
    position: 'absolute', top: 95, left: 70, width: 100, height: 50,
    justifyContent: 'center', alignItems: 'center',
  },
  centerText: { fontSize: 12, fontFamily: 'Cinzel_700Bold', textAlign: 'center' },
  attrNode: { position: 'absolute', alignItems: 'center' },
  nodeValue: { fontSize: 22, fontFamily: 'Oswald_700Bold' },
  nodeLabel: { fontSize: 8, fontFamily: 'Oswald_400Regular' },
  nodeAttr: { fontSize: 10, fontFamily: 'Oswald_700Bold' },
  // Posições (ajuste conforme seu pentagrama)
  agiPos: { top: 5,   left: 95  },
  intPos: { top: 75,  right: 5  },
  vigPos: { bottom: 5, right: 30 },
  prePos: { bottom: 5, left: 30 },
  forPos: { top: 75,  left: 5   },
});
```

---

## PASSO 13 — TELA DE PERÍCIAS

```typescript
// app/character/[id]/pericias.tsx
import { FlatList, View, Text, StyleSheet, TouchableOpacity, ImageBackground } from 'react-native';
import { useLocalSearchParams } from 'expo-router';
import { useCharacterStore } from '@/store/characterStore';
import { useThemeStore } from '@/store/themeStore';
import { THEMES } from '@/constants/themes';
import { PERICIAS_CONFIG, getBonusAtributo, BONUS_TREINAMENTO } from '@/constants/ordemData';
import { PericiaKey } from '@/types/character';

export default function PericiasScreen() {
  const { id } = useLocalSearchParams<{ id: string }>();
  const char = useCharacterStore((s) => s.characters.find((c) => c.id === id))!;
  const { togglePericiaTreinada } = useCharacterStore();
  const { theme } = useThemeStore();
  const T = THEMES[char?.tema ?? theme];

  const periciasList = Object.entries(PERICIAS_CONFIG);

  return (
    <ImageBackground source={T.bgImage} style={{ flex: 1 }}>
      <View style={{ flex: 1, backgroundColor: 'rgba(0,0,0,0.80)' }}>
        <Text style={[styles.title, { color: T.primary }]}>Perícias</Text>

        {/* Cabeçalho da tabela */}
        <View style={styles.headerRow}>
          <Text style={[styles.headerCell, styles.nameCell, { color: T.primary }]}>Nome</Text>
          <Text style={[styles.headerCell, { color: T.primary }]}>Dado</Text>
          <Text style={[styles.headerCell, { color: T.primary }]}>Bônus</Text>
          <Text style={[styles.headerCell, { color: T.primary }]}>Treino</Text>
          <Text style={[styles.headerCell, { color: T.primary }]}>Total</Text>
        </View>

        <FlatList
          data={periciasList}
          keyExtractor={([k]) => k}
          renderItem={({ item: [key, config] }) => {
            const status = char.pericias[key as PericiaKey];
            const attrVal = char.atributosBase[config.atributo];
            const bonusAttr = getBonusAtributo(attrVal);
            const bonusTreinamento = status?.treinada ? BONUS_TREINAMENTO.treinada : 0;
            const total = bonusAttr + bonusTreinamento + (status?.bonus ?? 0);

            return (
              <View style={[styles.periciaRow, { borderBottomColor: '#222' }]}>
                {/* Nome + marcadores */}
                <Text style={[styles.periciaName, { color: T.text }]}>
                  {config.nome}{config.marcadores ? config.marcadores : ''}
                </Text>

                {/* Atributo */}
                <View style={[styles.cell, styles.attrBox, { borderColor: T.border }]}>
                  <Text style={{ color: T.text, fontSize: 11 }}>{config.atributo}</Text>
                </View>

                {/* Bônus atributo */}
                <View style={[styles.cell, styles.bonusBox, { borderColor: T.border }]}>
                  <Text style={{ color: T.text }}>+{bonusAttr}</Text>
                </View>

                {/* Treinamento (toggle) */}
                <TouchableOpacity
                  onPress={() => togglePericiaTreinada(id, key as PericiaKey)}
                  style={[styles.cell, styles.treinoBox, {
                    backgroundColor: status?.treinada ? T.primary : 'transparent',
                    borderColor: T.border,
                  }]}
                >
                  {status?.treinada && (
                    <Text style={{ color: '#FFF', fontSize: 12 }}>✓</Text>
                  )}
                </TouchableOpacity>

                {/* Total */}
                <View style={[styles.cell, styles.totalBox, { borderColor: T.border }]}>
                  <Text style={{ color: T.primary, fontFamily: 'Oswald_700Bold' }}>+{total}</Text>
                </View>
              </View>
            );
          }}
        />
      </View>
    </ImageBackground>
  );
}

const styles = StyleSheet.create({
  title: { fontSize: 22, fontFamily: 'Cinzel_700Bold', textAlign: 'center', paddingTop: 60, marginBottom: 8 },
  headerRow: { flexDirection: 'row', paddingHorizontal: 12, paddingVertical: 4, borderBottomWidth: 1, borderBottomColor: '#333' },
  headerCell: { fontSize: 11, fontFamily: 'Oswald_700Bold', textAlign: 'center' },
  nameCell: { flex: 1, textAlign: 'left' },
  periciaRow: { flexDirection: 'row', alignItems: 'center', paddingHorizontal: 12, paddingVertical: 6, borderBottomWidth: 1 },
  periciaName: { flex: 1, fontSize: 13, fontFamily: 'Oswald_400Regular' },
  cell: { borderWidth: 1, borderRadius: 4, justifyContent: 'center', alignItems: 'center' },
  attrBox: { width: 36, height: 26, marginHorizontal: 2 },
  bonusBox: { width: 32, height: 26, marginHorizontal: 2 },
  treinoBox: { width: 26, height: 26, marginHorizontal: 2 },
  totalBox: { width: 36, height: 26, marginHorizontal: 2 },
});
```

---

## PASSO 14 — UTILITÁRIOS

```typescript
// src/utils/helpers.ts
import { Pericias, PericiaKey } from '@/types/character';
import { PERICIAS_CONFIG } from '@/constants/ordemData';

export function createDefaultPericias(pericias_treinadas: string[] = []): Pericias {
  const result = {} as Pericias;
  (Object.keys(PERICIAS_CONFIG) as PericiaKey[]).forEach((key) => {
    result[key] = {
      treinada: pericias_treinadas.includes(key),
      bonus: 0,
    };
  });
  return result;
}

export function generateId(): string {
  return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
}

export function formatDate(iso: string): string {
  return new Date(iso).toLocaleDateString('pt-BR');
}
```

---

## PASSO 15 — TELA HOME (Lista de personagens)

```typescript
// app/index.tsx
import { View, Text, FlatList, TouchableOpacity, StyleSheet, ImageBackground } from 'react-native';
import { useRouter } from 'expo-router';
import { useCharacterStore } from '@/store/characterStore';
import { useThemeStore } from '@/store/themeStore';
import { THEMES } from '@/constants/themes';
import { Ionicons } from '@expo/vector-icons';

export default function HomeScreen() {
  const router = useRouter();
  const { characters, setActiveCharacter } = useCharacterStore();
  const { theme } = useThemeStore();
  const T = THEMES[theme];

  function openCharacter(id: string) {
    setActiveCharacter(id);
    router.push(`/character/${id}/profile`);
  }

  return (
    <ImageBackground source={T.bgImage} style={{ flex: 1 }}>
      <View style={{ flex: 1, backgroundColor: 'rgba(0,0,0,0.85)' }}>
        <Text style={[styles.title, { color: T.primary }]}>Ordem Paranormal</Text>
        <Text style={[styles.subtitle, { color: T.textSecondary }]}>Seus Personagens</Text>

        <FlatList
          data={characters}
          keyExtractor={(c) => c.id}
          contentContainerStyle={styles.list}
          ListEmptyComponent={
            <View style={styles.empty}>
              <Text style={{ color: '#555', textAlign: 'center' }}>
                Nenhum personagem criado.{'\n'}Clique no + para começar.
              </Text>
            </View>
          }
          renderItem={({ item: char }) => (
            <TouchableOpacity
              onPress={() => openCharacter(char.id)}
              style={[styles.card, { borderColor: THEMES[char.tema].primary }]}
            >
              <View style={[styles.cardAccent, { backgroundColor: THEMES[char.tema].primary }]} />
              <View style={styles.cardContent}>
                <Text style={[styles.cardName, { color: '#FFF' }]}>{char.nome}</Text>
                <Text style={[styles.cardInfo, { color: '#AAA' }]}>
                  {char.classe} • {char.origem} • NEX {char.nex}%
                </Text>
                <Text style={[styles.cardInfo, { color: THEMES[char.tema].primary }]}>
                  PV: {char.pv.atual}/{char.pv.total} | PE: {char.pe.atual}/{char.pe.total}
                </Text>
              </View>
            </TouchableOpacity>
          )}
        />

        {/* Botão criar personagem */}
        <TouchableOpacity
          onPress={() => router.push('/create')}
          style={[styles.fab, { backgroundColor: T.primary }]}
        >
          <Ionicons name="add" size={32} color="#FFF" />
        </TouchableOpacity>
      </View>
    </ImageBackground>
  );
}

const styles = StyleSheet.create({
  title: { fontSize: 28, fontFamily: 'Cinzel_700Bold', textAlign: 'center', paddingTop: 70 },
  subtitle: { textAlign: 'center', marginBottom: 20, fontFamily: 'Oswald_400Regular' },
  list: { paddingHorizontal: 20, gap: 12 },
  empty: { marginTop: 80 },
  card: { flexDirection: 'row', borderWidth: 1, borderRadius: 8, overflow: 'hidden', backgroundColor: '#111' },
  cardAccent: { width: 6 },
  cardContent: { flex: 1, padding: 12 },
  cardName: { fontSize: 18, fontFamily: 'Oswald_700Bold' },
  cardInfo: { fontSize: 12, marginTop: 2, fontFamily: 'Oswald_400Regular' },
  fab: { position: 'absolute', bottom: 30, right: 24, width: 60, height: 60, borderRadius: 30, justifyContent: 'center', alignItems: 'center', elevation: 6 },
});
```

---

## PASSO 16 — RODAR O PROJETO

```bash
# Desenvolvimento
npx expo start

# iOS (Simulador ou dispositivo físico via Expo Go)
npx expo run:ios

# Android
npx expo run:android

# Build de produção (EAS Build)
npm install -g eas-cli
eas login
eas build:configure
eas build --platform android --profile preview
```

---

## 📋 CHECKLIST DE FUNCIONALIDADES AUTOMÁTICAS

| Funcionalidade | Como funciona |
|---|---|
| **PV Total** | Recalcula ao mudar VIG, NEX ou Classe |
| **PE Total** | Recalcula ao mudar PRE, NEX ou Classe |
| **Sanidade Total** | Recalcula ao mudar PRE |
| **DEF** | Recalcula ao mudar AGI ou adicionar armadura |
| **Limite de Itens** | Recalcula ao mudar FOR |
| **Carga Máxima** | Recalcula ao mudar FOR |
| **DT de Rituais** | Recalcula ao mudar NEX ou INT |
| **Bônus de Perícia** | Atualiza em tempo real (atributo + treino + extra) |
| **Dados de Perícia** | Mostra atributo correto por perícia |
| **Equipamentos iniciais** | Carregados automaticamente pela Origem |
| **Perícias da Origem** | Marcadas como treinadas automaticamente |
| **Bônus de Atributo da Origem** | Somados ao criar o personagem |
| **Patente** | Derivada do NEX automaticamente |

---

## 🎨 ASSETS — RENOMEAR E COLOCAR EM `assets/images/`

```
b1_2.png  →  bg_red.png
b1_3.png  →  bg_blue.png
b1_4.png  →  bg_purple.png
Group_1.png  →  pentagrama.png
Group_4.png  →  sigil_red.png    (ou logo para o tema vermelho)
Group_7.png  →  sigil_blue.png
Group_10.png →  sigil_purple.png
Group_13.png →  logo_op.png
```

---

## 🔮 PRÓXIMOS PASSOS SUGERIDOS

1. **Tela de Inventário completa** com categorias (0-3), peso total, compra/venda
2. **Tela de Habilidades** com pesquisa por nome e filtro por classe/trilha
3. **Tela de Rituais** com DT, custo em PE, elementos (Fogo, Água, Terra, Ar, Sangue, Morte, Mente)
4. **Rolagem de dados embutida** — toque na perícia e rola d20+bônus com animação
5. **Gerenciador de combate** — iniciativa, HP inimigos, rodadas
6. **Exportar PDF** da ficha usando expo-print
7. **Multi-personagem** — trocar sem sair do app
8. **Foto do personagem** via expo-image-picker
9. **Backup em nuvem** via Supabase ou Firebase
10. **Modo offline completo** (já implementado com MMKV)
```

// src/store/characterStore.ts

// ❌ REMOVA isso:
import { MMKV } from 'react-native-mmkv';

const storage = new MMKV({ id: 'ordem-paranormal-store' });

const zustandStorage = {
  getItem: (name: string) => storage.getString(name) ?? null,
  setItem: (name: string, value: string) => storage.set(name, value),
  removeItem: (name: string) => storage.delete(name),
};

// ✅ COLOQUE isso:
import AsyncStorage from '@react-native-async-storage/async-storage';

const zustandStorage = createJSONStorage(() => AsyncStorage);