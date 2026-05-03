---
trigger: always_on
description: This file provides guidance to WARP (warp.dev) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## 第一性原理

请使用第一性原理思考。你不能总是假设我非常清楚自己想要什么和该怎么得到。请保持审慎，从原始需求和问题出发，如果动机和目标不清晰，停下来和我讨论。

## 方案规范

当需要你给出修改或重构方案时必须符合以下规范：

- 不允许给出兼容性或补丁性的方案
- 不允许过度设计，保持最短路径实现且不能违反第一条要求

## Commands

### Local setup & development

- `npm install` — install dependencies
- `npm run dev` — start dev server (auto-detected locale)
- `npm run dev:secure` — dev server in secure mode (`--mode secure`, respects `VITE_ENABLE_SECURE_MODE`)
- `npm run fetch-data` — update game data & icons under `public/assets` (run before build if upstream data changes or if local data is missing)

### Build, preview, and deploy

- `npx vue-tsc -b` — type-check the project
- `npm run build` — production build (type-check + Vite build + generate `dist/en/index.html`). Note: `prebuild` runs `fetch-data` automatically.
- `npm run build:secure` — production build for secure deployment (adds `--mode secure`, runs SEO cleanup)
- `npm run preview` — preview the built site locally (serves `dist`)
- GitHub Pages: push to `main` triggers `.github/workflows/deploy.yml` (sets `VITE_BASE_PATH=/BuildingMomo/`)
- Cloudflare: `npm run deploy:cloudflare` (uses `build:secure`)
- Netlify: `npm run deploy:netlify`

### Formatting

- Prettier is the sole formatter. Config in `.prettierrc.json`: no semicolons, single quotes, 100-char print width, 2-space indent, `prettier-plugin-tailwindcss`.
- Husky pre-commit hook runs `lint-staged` → `prettier --write` on staged `js/jsx/ts/tsx/vue/json/md` files.
- Format manually: `npx prettier --write <file>`

### Tests

- **No test runner or `npm test` script is configured** in this repository as of now.

## Code style & conventions

- Path alias: `@/` maps to `src/` (configured in both `tsconfig.json` and `vite.config.ts`)
- TypeScript strict mode is enabled (`tsconfig.app.json`)
- UI components use **shadcn-vue** (new-york style). Config in `components.json`; add components via `npx shadcn-vue@latest add <component>`. Primitives live in `src/components/ui/`.
- Icons: `lucide-vue-next`
- CSS: Tailwind CSS v4 via `@tailwindcss/vite` plugin; base styles in `src/index.css`
- Vite is overridden to `rolldown-vite` in `package.json` overrides
- Three.js is deduped in `vite.config.ts` resolve to avoid multi-instance issues

## High-level architecture

### Overview

- Single-page web application (Vue 3 + Vite) providing a 3D visual editor for **Infinity Nikki** home building schemes.
- The core domain model is a "home scheme" representing furniture instances loaded from / exported to the game's `BUILD_SAVEDATA_*.json` files.
- State is managed centrally with **Pinia** stores; rendering uses **Three.js** via **TresJS** (`@tresjs/core` and `@tresjs/cientos`) with heavy use of **instanced meshes** for performance.

### Application shell

- Entry point: `src/main.ts` creates the Vue app, installs Pinia, and mounts `src/App.vue`.
- `src/App.vue` is the top-level layout:
  - Sets up theme handling (light/dark/auto) based on `settingsStore` and media queries.
  - Restores workspace state via `useWorkspaceWorker` (IndexedDB + Web Worker) when auto-save is enabled and a previous session marker exists.
  - Wires global keyboard shortcuts via `useKeyboardShortcuts`, delegating to the command system.
  - Overall structure: `Toolbar` at top, central area hosting either `WelcomeScreen`, `ThreeEditor` (for scheme tabs), or `DocsViewer` (for in-app docs), with `Sidebar` on the right and `StatusBar` at the bottom.
  - Global overlays: `Toaster` (vue-sonner), `CoordinateDialog`, `GlobalAlertDialog`.

### State management (Pinia stores)

#### `editorStore` – schemes and scene graph

- Defines the core **multi-document model**:
  - `schemes`: array of `HomeScheme` objects, each wrapping a scheme's metadata and items using `Ref`/`ShallowRef` for performance.
  - `activeSchemeId` and computed `activeScheme` to track the current working scheme.
  - Each scheme holds:
    - `items: ShallowRef<AppItem[]>` – flat list of furniture instances.
    - `selectedItemIds: ShallowRef<Set<string>>` – selection set.
    - `maxInstanceId`, `maxGroupId` – monotonically increasing allocators.
    - View-related state: `currentViewConfig`, `viewState` (camera/zoom preset snapshot).
    - `groupOrigins: ShallowRef<Map<number, string>>` – maps groupId to an origin item ID used for move/rotate operations.
    - `history` – undo/redo state tracking (managed by `useEditorHistory`).
- Derived indices for fast lookup:
  - `itemsMap` (internalId → AppItem) and `groupsMap` (groupId → Set<internalId>). These are used heavily by editor composables and the renderer.
- Scene/selection versioning:
  - `sceneVersion` and `selectionVersion` are numeric counters incremented by `triggerSceneUpdate` / `triggerSelectionUpdate` so that performance-critical observers (renderer, worker, validation store) can respond without deeply watching arrays/Sets.
  - `transactionVersion` is incremented whenever a new transaction is recorded, driving cloud synchronization (`useCloudSchemeSync`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChanIok/BuildingMomo](https://github.com/ChanIok/BuildingMomo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
