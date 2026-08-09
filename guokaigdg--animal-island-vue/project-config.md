---
trigger: always_on
description: A Vue 3 + TypeScript + Less component library inspired by Animal Crossing. Warm earth-tone palette + mint teal primary, big rounded pill shapes, game-button 3D shadows, gentle animations.
---

# animal-island-vue — Cursor Project Rules

A Vue 3 + TypeScript + Less component library inspired by Animal Crossing. Warm earth-tone palette + mint teal primary, big rounded pill shapes, game-button 3D shadows, gentle animations.

## 1. Tech Stack

- **Framework**: Vue 3.5+ (Composition API, `<script setup lang="ts">`)
- **Language**: TypeScript 5.7 (strict)
- **Build**: Vite 7 (library mode for the package, docs mode for the demo site)
- **Styles**: Less + CSS variables (NO CSS Modules, NO Tailwind)
- **Type check**: `vue-tsc --noEmit`
- **Demo**: separate `vite.config.docs.ts`; demos live in `demo/pages/`
- **Package entry**: `src/index.ts` (named exports of components + types)
- **Testing**: Vitest + `@vue/test-utils`

## 2. File Structure (MUST follow)

```
src/components/<Name>/
├── <Name>.vue          # SFC: <script setup lang="ts"> + <template> + <style lang="less" scoped>
├── types.ts            # optional, when types are shared across files or generic
└── index.ts            # re-exports the component + types
```

- New components MUST live under `src/components/<Name>/`. Never put components at `src/` root.
- `src/index.ts` MUST export the component **and** its prop types:
  ```ts
  export { default as Foo } from './components/Foo';
  export type { FooProps } from './components/Foo';
  ```
- Demo for every component MUST be created at `demo/pages/<Name>Demo.vue`, registered in `demo/router.ts` (or `ComponentPage.vue`), and listed in `demo/pageInfo.ts` with `{ title, desc }`.

## 3. Coding Style

- All SFCs use `<script setup lang="ts">`. Do NOT use Options API.
- Props with generics MUST be declared inline via `defineProps<{...}>()`. Do NOT also declare a named `interface Props` (triggers `TS4082` in `vite-plugin-dts`).
- Non-generic props can live in a separate `types.ts`.
- All props MUST have JSDoc comments (Chinese OK).
- Controlled values use `v-model` / `v-model:open` / `v-model:expanded` (`modelValue` + `update:modelValue`). Also support `defaultValue` for uncontrolled initial value.
- React's `ReactNode` props → Vue named slots (`#icon`, `#prefix`, `#suffix`, `#footer`, `#checked`, `#unchecked`, `#title`, `#empty`, Tabs/Table dynamic slots, etc.). Use the default slot for content that is just structured body.
- Use `v-for` + `:key` for list rendering. Never use `.map()` in a template.
- Use `defineEmits<{ (e: 'name', payload: T): void }>()` with typed payloads.
- Use `defineSlots<{ default?: () => unknown; icon?: () => unknown }>()` when slots matter.
- For Timer / one-shot intervals: use `onMounted` + `setInterval`, clean up in `onUnmounted`.

## 4. Styling Rules (CRITICAL)

- Every SFC uses `<style lang="less" scoped>` with **BEM** class names. **No CSS Modules (`*.module.less`). No Tailwind.**
- Block class: `.animal-<name>` (kebab-case). Modifier: `.animal-<name>--<mod>`. Element: `.animal-<name>__<elem>`.
- Import global tokens at the top of the `<style>` block:
  ```less
  @import '@/styles/variables.less';
  ```
  and reference them as `@bg-color-content`, `@text-color-body`, `@border-radius-base`, `@motion-ease`, etc.
- Never hardcode hex values that already exist as Less tokens. Hardcoded hex is only acceptable for one-off palette entries (e.g. Card `color` variants) defined in `variables.less`.
- **Shadow system is asymmetric — do not over-apply:**
  - `Button` `type="primary"` / `danger + primary` → 3D pixel-stack shadow `0 5px 0 0 #bdaea0` (hover +1, active -1).
  - `Button` `default` / `dashed` / `text` / `link` → soft elevation only (`0 2px 4px / 0 3px 10px rgba(61,52,40,...)`), no pixel stack.
  - `Input` → no shadow by default; only when `shadow={true}` opt-in, then `0 Npx 0 0 #d4c9b4`. Status (error/warning) shadows render regardless.
  - `Switch` → no outer box-shadow anywhere; only INSET shadow on the track. The handle has only a 2.5px border, NO `box-shadow`.
  - `Card` → NO `box-shadow`. Floats on hover via `transform: translateY(-2px)` only.
  - `Modal` → uses an SVG `clip-path: url(#animal-modal-clip)` blob, NOT a rounded rectangle. Never replace it.
  - `Title` (ribbon) → swallowtail clip-path + CSS triangle folds + 3deg perspective. Never render it as a plain pill/rect.
- **Cursor component** (`src/components/Cursor/Cursor.vue`) MUST use a non-scoped `<style>` block (or a global CSS file) because `scoped` selectors cannot pierce slot content. Use `:global(.animal-cursor)` or a top-level `<style>` without `scoped`.
- **Loading component** uses `position: absolute`, NOT `fixed` — it inherits the nearest positioned ancestor.

## 5. Design Tokens (cheat sheet)

| Token | Value | Use |
|---|---|---|
| `@primary-color` | `#19c8b9` | mint teal — brand primary, focus for buttons |
| `@primary-color-hover` | `#3dd4c6` | |
| `@primary-color-active` | `#11a89b` | |
| `@bg-color` | `#f8f8f0` | page background |
| `@bg-color-content` | `rgb(247, 243, 223)` | inside Card/Modal/Table |
| `@bg-color-input` | `rgb(247, 243, 223)` | |
| `@text-color` | `#794f27` | heading / sidebar text |
| `@text-color-body` | `#725d42` | in-component body text |
| `@text-color-muted` | `#8a7b66` | modal body / description |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [guokaigdg/animal-island-vue](https://github.com/guokaigdg/animal-island-vue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
