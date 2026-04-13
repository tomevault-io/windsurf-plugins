---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A component catalog (shadcn-vue style) for Clínica Adventista GoodHope. It is a **documentation site** — not a traditional app — where UI components are displayed, demonstrated, and documented so developers can copy them into other projects.

## Commands

```bash
npm run dev      # Start dev server at http://localhost:5173
npm run build    # Type-check with vue-tsc, then build
npm run preview  # Preview production build
```

There are no test scripts configured.

## Architecture

### Directory Structure

```
src/
├── components/
│   ├── ui/              # Reusable UI components (shadcn-vue pattern)
│   ├── common/          # Composite custom components
│   └── command/         # Command palette (CommandMenu.vue)
├── docs/
│   ├── components/      # Per-component documentation pages
│   │   └── [name]/      # [Name]Docs.vue + [Name]Examples.vue
│   ├── shared/          # Shared doc utilities: CodeBlock.vue, DocExampleContainer.vue
│   └── snippets/        # TS files exporting raw code strings for display
├── utils/
│   ├── components.ts    # Central registry of all nav sections + component list
│   └── config.ts        # Site-level config (siteConfig, navItems)
├── router/index.ts      # Vue Router setup
├── views/               # Page-level views
│   └── ComponentDetailView.vue  # Dynamic component doc loader via docsRegistry
├── lib/utils.ts         # cn() and valueUpdater() helpers
└── style.css            # Global CSS + CSS custom properties for design tokens
```

### How the Component Catalog Works

The app is a docs site with a sidebar + router. When you navigate to `/docs/components/:name`:

1. `ComponentDetailView.vue` uses a `docsRegistry` map (`name → DocsComponent`) to dynamically render the right `*Docs.vue`
2. Each `*Docs.vue` uses `DocExampleContainer` (shows live example + syntax-highlighted code) and imports the actual UI component from `@/components/ui/`
3. Navigation is driven by `src/utils/components.ts` — the `components` array is the single source of truth for which components appear in the sidebar

### Adding a New Component

Follow these steps in order:

1. **Create** `src/components/ui/[name]/[Name].vue` + `src/components/ui/[name]/index.ts`
   - `index.ts` exports the component and defines CVA variants
2. **Create** `src/docs/components/[name]/[Name]Docs.vue` + `[Name]Examples.vue`
   - Use `DocExampleContainer` and import code from snippets
3. **Create** `src/docs/snippets/[name]Examples.ts` — exports raw code strings
4. **Register** in `src/utils/components.ts` — add entry to the `components` array
5. **Register** in `src/views/ComponentDetailView.vue` — import `[Name]Docs` and add to `docsRegistry`

### Component Pattern (CVA + Composition API)

All UI components follow this structure:

**`index.ts`** — exports component + CVA variant function:
```typescript
import type { VariantProps } from 'class-variance-authority'
import { cva } from 'class-variance-authority'

export { default as MyComponent } from './MyComponent.vue'

export const myComponentVariants = cva('base-classes', {
  variants: {
    variant: { default: '...', secondary: '...', tertiary: '...' },
    size: { sm: '...', default: '...', lg: '...' },
  },
  defaultVariants: { variant: 'default', size: 'default' },
})

export type MyComponentVariants = VariantProps<typeof myComponentVariants>
```

**`MyComponent.vue`** — uses `<script setup lang="ts">`, `Primitive` from reka-ui when needed, `cn()` for class merging:
```vue
<script setup lang="ts">
import type { HTMLAttributes } from 'vue'
import { cn } from '@/lib/utils'
import { myComponentVariants } from '.'
import type { MyComponentVariants } from '.'

interface Props {
  variant?: MyComponentVariants['variant']
  class?: HTMLAttributes['class']
}
const props = withDefaults(defineProps<Props>(), { variant: 'default' })
</script>

<template>
  <div :class="cn(myComponentVariants({ variant }), props.class)">
    <slot />
  </div>
</template>
```

## Design System

### Brand Colors (Soul MV palette — the only allowed colors)

| Token | Tailwind class prefix | Value | Description |
|---|---|---|---|
| Primary | `primary` | `#4672b1` | Blue (Pantone 7684C) |
| Secondary | `secondary` | `#0ca4eb` | Sky Blue (Process Blue C) |
| Tertiary | `tertiary` | `#1ab5c0` | Teal |

Use semantic tokens (`bg-primary text-primary-foreground`) — never arbitrary hex values like `bg-[#123456]`.

CSS custom properties are defined in `src/style.css` as HSL values (e.g. `--primary: 216 47% 48%`).

### shadcn-vue Configuration

- Style: `new-york`
- Icon library: `lucide` (lucide-vue-next)
- Path aliases: `@/components/ui` → `@/components/ui`, `@/lib/utils` → `@/lib/utils`

## Code Conventions

- Always use `<script setup lang="ts">` (never Options API)
- No `<style scoped>` — use Tailwind utility classes with `cn()` for all styling
- Import order: types → vue core → external libs → internal `@/` imports
- Avoid `any` types; use explicit TypeScript interfaces for all props
- File naming: components in `PascalCase.vue`, utilities in `camelCase.ts`

## Branching Strategy (Ship/Show/Ask)

- **Ship** — commit directly to `main` for trivial/low-risk changes
- **Show** — create branch `show/description`, merge immediately, async review
- **Ask** — create branch `ask/description`, wait for approval before merge

Commit format: `feat(scope): descripción` using Conventional Commits.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/luis-keny)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/luis-keny)
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
