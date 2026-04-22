---
trigger: always_on
description: Regras específicas do projeto Yooga - Mantra
---


# CONTEXTO @YOOGA-TECNOLOGIA - MANTRA DESIGN SYSTEM

## Princípios Fundamentais

- **DRY**: Não se repetir, centralizar lógica comum em tokens e utilities
- **SOLID**: Single Responsibility, Open/Closed, Liskov, Interface Segregation, Dependency Inversion
- **KISS**: Manter simples, fazer o mais simples que funcione
- **YAGNI**: Implementar apenas o necessário agora
- **Clean Code**: Código autoexplicativo, sem comentários redundantes
- **Consistência**: Seguir padrões estabelecidos no projeto

## Regras Gerais

### Comunicação

- Responder **SEMPRE** em PT-BR
- Não se desculpar ou agradecer
- Comunicar de forma clara, direta, sem floreios
- Fornecer respostas concisas e relevantes

### Processo

- **Nunca quebrar funcionalidades existentes**
- Buscar menor diff possível
- Mudanças incrementais, arquivo por arquivo
- Documentar mudanças nas mensagens de commit
- Solicitar print do resultado e fornecer análise detalhada
- Implementar testes unitários quando criar funcionalidades novas

### Qualidade de Código

- **EVITAR** adicionar comentários redundantes ao código
- Preservar estruturas de código existentes
- Verificar informações antes de fazer mudanças
- Nunca adicionar placeholders ou TODOs
- Entregar código pronto para produção

## Estrutura do Projeto Mantra

### Arquitetura

```md
src/
├── components/          # Componentes StencilJS
│   └── [component]/
│       ├── [component].tsx      # Lógica do componente
│       ├── [component].types.ts # Definições de tipos
│       ├── [component].scss      # Estilos (barrel file)
│       ├── [component].spec.tsx # Testes unitários
│       ├── [component].stories.ts # Storybook
│       ├── readme.md            # Documentação
│       └── styles/              # Arquivos SCSS (opcional)
│           ├── __base.scss
│           ├── _variant-*.scss
│           └── index.scss
├── shared/
│   ├── theme/
│   │   ├── core/               # Funções e mixins
│   │   └── tokens/             # Design tokens
│   │       ├── actions/        # Tokens de ações (botões)
│   │       ├── feedbacks/      # Tokens de feedback
│   │       ├── inputs/         # Tokens de inputs
│   │       ├── _primitives.scss
│   │       └── _sizing.scss
│   └── assets/fonts/            # Fontes
└── utils/                      # Funções utilitárias
```

## Convenções de Nomenclatura

### Arquivos

- **Componentes**: `kebab-case` (ex: `field-number.tsx`)
- **Tipos**: `[component].types.ts`
- **Estilos**: `[component].scss` ou `styles/__base.scss`
- **Testes**: `[component].spec.tsx`
- **Stories**: `[component].stories.ts`

### Classes CSS

- **BEM-like**: `.mnt-[component]-[element]-[modifier]`
- Exemplo: `.mnt-button-icon-regular`
- Prefixo: `mnt-` (Mantra)

### Variáveis SCSS

- **Componentes**: `$component-prefix: 'mnt-[component]';`
- **Tokens**: `$[token-name]-map: ( ... );`
- **Configurações**: `$[size/variant]-configs: ( ... );`

### TypeScript

- **Classes**: `PascalCase` (ex: `export class Button`)
- **Props**: `camelCase` (ex: `@Prop() iconLeft`)
- **Interfaces**: `PascalCase` + `Props` (ex: `ButtonProps`)
- **Tipos**: `PascalCase` (ex: `IconSize`)

## Estrutura de Componentes StencilJS

### Arquivos Obrigatórios

1. **`[component].tsx`** - Componente principal

```ts
   @Component({
     tag: 'mnt-[component]',
     styleUrl: '[component].scss',
     shadow: false,
   })
   export class ComponentName {
     @Prop() propName: type;

     render() {
       return (
         // JSX
       );
     }
   }
   ```

2. **`[component].types.ts`** - Definições de tipos
   ```typescript
   export const componentPrefix = 'component';

   export interface ComponentBaseProps {
     size?: 'small' | 'medium' | 'large';
     color?: string;
   }

   export interface ComponentProps extends ComponentBaseProps {
     // props específicas
   }
   ```

3. **`[component].scss`** - Estilos (barrel file)
   ```scss
   /**
    * This file is a barrel file for the styles of <mnt-component>.
    * WARNING: Do not add CSS rules directly here.
    * Use the variant files in styles/ directory.
    **/

   @use './styles';
   ```

### Arquivos Opcionais

4. **`styles/`** - Estrutura modular de estilos
   - `__base.scss` - Variáveis e estilos base
   - `_variant-*.scss` - Variantes específicas
   - `index.scss` - Barrel file que importa todos

5. **`[component].stories.ts`** - Documentação Storybook
   ```typescript
   const meta: Meta<ComponentProps> = {
     title: 'Category/Component',
     component: 'mnt-component',
     argTypes: { ... },
   };

   export default meta;
   ```

6. **`[component].spec.tsx`** - Testes unitários

7. **`readme.md`** - Documentação (gerada automaticamente)

## Organização de Estilos (SCSS)

### Structure Pattern (Badge, Button, Field-Number)

Quando um componente tem **múltiplas variantes ou tamanhos**:

```scss
// styles/index.scss (barrel)
@forward './__base';
@use './_variant-plain';
@use './_variant-default';
@use './_variant-simple';

// styles/__base.scss (variáveis e base)
@use 'sass:map';
@use '../../../shared/theme/core/theme' as theme;

$component-prefix: theme.get-prefix('component');

// Variáveis de esquema de cores
$component-color-scheme: (
  default: (...),
  hover: (...),
  focus: (...),
);

// Base styles
mnt-component {
  // base
}


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yooga-tecnologia) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
