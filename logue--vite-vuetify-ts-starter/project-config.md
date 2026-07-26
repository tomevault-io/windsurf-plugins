---
trigger: always_on
description: This file provides guidance for AI coding agents (GitHub Copilot, Claude, Cursor, etc.) working in this repository.
---

# AGENT.md

This file provides guidance for AI coding agents (GitHub Copilot, Claude, Cursor, etc.) working in this repository.

---

## Project Overview

**Vue 3 + Vuetify + TypeScript + Vite** starter template.

- **Framework**: Vue 3 (`<script setup>` SFC only)
- **Build tool**: Vite 8
- **Language**: TypeScript (strict)
- **State management**: Pinia (setup-store style)
- **Routing**: Vue Router 5
- **Package manager**: pnpm (do not use npm or yarn)
- **Node version**: `>=22.13.0`
- **UI library**: Vuetify 4

---

## Commands

```bash
pnpm dev              # Start dev server (http://localhost:5173)
pnpm build            # Type-check + production build
pnpm lint             # Run all linters (oxlint → eslint → prettier → stylelint)
pnpm type-check       # vue-tsc type check only
pnpm test:unit        # Vitest unit tests
pnpm test:coverage    # Vitest with coverage report
pnpm test:e2e         # Playwright E2E tests
pnpm build:analyze    # Bundle size analysis (rollup-plugin-visualizer)
pnpm clean            # Clear Vite dev cache
```

Always run `pnpm lint` and `pnpm type-check` before committing. These are also enforced by husky pre-commit hooks via lint-staged.

---

## TypeScript Rules

- **No `any`** — use `unknown` and narrow with type guards.
- **Explicit return types** on exported functions (Pinia stores are exempt due to setup-style inference).
- **Use `type` over `interface`** for object shapes; extend via intersection (`&`).
- **Union literal types** instead of magic strings:
  ```ts
  type Status = 'active' | 'inactive' | 'pending';
  ```
- **Underscore prefix** for intentionally unused variables: `_value`, `_error`.
- **Array type syntax**: `string[]` not `Array<string>`.
- **Generic constructors**: left-hand side style — `const map: Map<string, User> = new Map()`.

---

## Vue SFC Rules

### Script

- Always use `<script setup lang="ts">` — Options API is prohibited.
- `defineProps` and `defineEmits` must use **type-based declarations** (runtime declarations are prohibited):

  ```ts
  // OK
  const props = defineProps<{ title: string; count?: number }>();
  const emit = defineEmits<{ change: [value: string]; close: [] }>();

  // NG
  const props = defineProps({ title: String });
  ```

- Return values from composables as individual `ref`s (not `reactive`) to enable destructuring.
- Internal state exposed from composables should be wrapped in `readonly()`.

### Template

- **Self-closing void elements**: `<br />`, `<img />`, `<input />`.
- **Attribute order** (enforced by `vue/attributes-order`):
  `DEFINITION` → `LIST_RENDERING` → `CONDITIONALS` → `RENDER_MODIFIERS` → `UNIQUE` → `TWO_WAY_BINDING` → `OTHER_DIRECTIVES` → `ATTR_DYNAMIC` → `ATTR_STATIC` → `ATTR_SHORTHAND_BOOL` → `EVENTS` → `CONTENT`
- Run `pnpm lint` to auto-fix attribute order.

### Style

- Always use `<style lang="scss" scoped>` — unscoped styles are prohibited.
- CSS custom properties (design tokens) must be defined in a shared file (e.g., `src/styles/variables.scss`) and not duplicated per component.
- CSS property order is enforced by stylelint-order — run `pnpm lint:style` to auto-fix.

---

## Component Naming

- Component files: **PascalCase**, multi-word required (e.g., `UserCard.vue`, `AppHeader.vue`).
  - `src/components/**/*.vue` — `error`
  - `src/pages/**/*.vue` and `src/layouts/**/*.vue` — `warn` (file-based routing constraint)
- Do not create single-word components like `Header.vue` or `Card.vue` outside of pages/layouts.

---

## Import Rules

- **Always use the `@/` alias** for internal imports — relative parent traversal (`../`) is prohibited in application code:

  ```ts
  // OK
  import { useUserStore } from '@/store/user';
  import type { User } from '@/types';

  // NG
  import { useUserStore } from '../../../store/user';
  ```

  > **Exception**: test files under `src/**/__tests__/` may use `../` to import the component under test (e.g., `import MyComponent from '../MyComponent.vue'`). This is intentional and the ESLint rule is disabled for that scope.

- The `~` alias maps to `node_modules` (e.g., `~/some-lib/style.css`).
- **Import order** (enforced by `import-x/order`, auto-fixed by `pnpm lint:eslint`):
  1. Node built-ins
  2. Vue core (`vue`, `vue-router`, `pinia`, `@vue/*`, `@vitejs/*`)
  3. External packages
  4. Internal (`@/**`)
  5. Sibling / index
  6. Type imports
     A blank line is required between each group.

---

## Pinia Store Rules

- Use **setup-store style** exclusively (not options-store style):
  ```ts
  // OK
  export const useUserStore = defineStore('user', () => {
    const user = ref<User | null>(null);
    function setUser(u: User) {
      user.value = u;
    }
    return { user, setUser };
  });
  ```
- Store ID must match the file name (e.g., `defineStore('user', ...)` in `store/user.ts`).
- Persist state via `pinia-plugin-persistedstate` — do not manually read/write `localStorage`.

---

## Accessibility (a11y)

Rules are enforced by `eslint-plugin-vuejs-accessibility`. Key points:

- `<a>` elements must have text content (`vuejs-accessibility/anchor-has-content: error`).
- Avoid `autofocus` (`vuejs-accessibility/no-autofocus: warn`).
- For **single input** → use `<label for="id">` or nesting.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [logue/vite-vuetify-ts-starter](https://github.com/logue/vite-vuetify-ts-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
