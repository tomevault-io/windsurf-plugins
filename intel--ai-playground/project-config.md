---
trigger: always_on
description: provides on-demand LLM inference (health: `GET /health`).
---

# AGENTS.md — AI Playground

Concise reference for AI coding agents working in this repository.

## Project Overview

Electron + Vue.js desktop app for AI inference on Intel GPUs. Multi-process architecture:
Electron main process orchestrates Vue.js frontend and multiple Python/native backend services
(AI Backend, ComfyUI, LlamaCPP, OpenVINO). Frontend code lives in `WebUI/`.

## Mandatory Rules

- Use **composition over inheritance** — never introduce new class hierarchies.
- Do **not** use classes unless extending an existing set of classes of the same type.
- Use **`type`** instead of `interface`, unless an interface is strictly necessary for implementation.

## Build / Dev / Test Commands

All commands run from the **`WebUI/`** directory.

```bash
# Install dependencies
npm install

# Start dev server + Electron
npm run dev

# Run all tests once
npm test

# Run tests in watch mode
npm run test:watch

# Run a single test file
npx vitest run electron/test/subprocesses/deviceArch.test.ts

# Run tests matching a name pattern
npx vitest run --testNamePattern "getDeviceArch"

# Lint (ESLint with auto-fix)
npm run lint

# Lint without auto-fix (CI mode)
npm run lint:ci

# Format (Prettier)
npm run format

# Format check only (CI mode)
npm run format:ci

# TypeScript type-check (no emit)
npx vue-tsc --noEmit

# Full production build (Windows installer)
npm run fetch-external-resources
npm run build
```

Python backend (`service/`) uses **Ruff** for linting (runs in CI via GitHub Actions).

## Test Conventions

- Framework: **Vitest 3.2+** with `node` environment.
- Test file pattern: `**/*.test.ts` (not `.spec.ts`).
- Path aliases: `@` → `./src`, `electron` → `./electron`.
- Tests use `describe` / `it` / `expect`. Mock Electron with `vi.mock('electron', ...)`.
- Tests live alongside source in `electron/test/` (currently unit tests for Electron main process only).

## Code Style

### Formatting (enforced by Prettier + EditorConfig)

- **No semicolons**
- **Single quotes**
- **2-space indentation** (spaces, not tabs)
- **100-character line width**
- **LF line endings**
- **Trailing whitespace trimmed**, final newline inserted

### TypeScript

- Target: **ES2023**, module: **ESNext** with bundler resolution.
- **Strict mode** enabled.
- Prefix unused variables/parameters with `_` (e.g., `_event`, `_unused`).
  Variables ending in `Schema` are also exempt from unused-var checks.
- Use `type` over `interface` (see Mandatory Rules above).

### Vue Components

- Always use `<script setup lang="ts">` with Composition API.
- Define props with `defineProps<{ ... }>()` using TypeScript generics.
- Define emits with `defineEmits<{ ... }>()` using TypeScript generics.
- File naming: **PascalCase** (`MyComponent.vue`).
- Single-word component names are allowed (`vue/multi-word-component-names` is off).

### Pinia Stores

- Use setup syntax: `defineStore('name', () => { ... })`.
- Enable persistence with `{ persist: true }` option where needed. Properties picked for persistence need to be returned, even if they are not used externally.
- Add HMR support: `if (import.meta.hot) import.meta.hot.accept(acceptHMRUpdate(...))`.
- Store files: **camelCase** in `WebUI/src/assets/js/store/` (e.g., `backendServices.ts`).
- Store hooks use `use` prefix: `useBackendServices`, `useTextInference`.
- Stores may import other stores for composition.
- **Store instantiation**: Always use a regular `import` at the top of the file and call `const someStore = useSomeStore()` at the top of the `defineStore` setup function or `<script setup>` block. **Never** use dynamic `import()` or inline `useSomeStore()` calls inside nested functions/callbacks.

### Import Ordering

No strict enforcement, but follow the prevailing convention:
1. External packages (`vue`, `pinia`, `zod`, `@ai-sdk/*`)
2. Internal stores (`@/assets/js/store/...`)
3. Components (`@/components/...`)
4. Utilities (`@/lib/utils`, `@/assets/js/toast`)

### Naming Conventions

| Element               | Convention   | Example                          |
|-----------------------|-------------|----------------------------------|
| Vue components/files  | PascalCase  | `ModelSelector.vue`              |
| Store files           | camelCase   | `backendServices.ts`             |
| Functions/variables   | camelCase   | `startService`, `currentStatus`  |
| Types                 | PascalCase  | `BackendStatus`, `ModelPaths`    |
| Store composables     | `use` prefix| `useBackendServices()`           |
| Backend service names | kebab-case  | `'ai-backend'`, `'comfyui-backend'` |
| Python modules        | snake_case  | `web_api.py`, `llm_biz.py`      |

### Error Handling

- Wrap async operations in `try/catch`.
- Log errors with `console.error()`.
- Show user-facing errors via toast: `import * as toast from '@/assets/js/toast'` then `toast.error(msg)`.
- IPC handlers return `{ success: boolean, error?: string }` pattern for error propagation.
- Python backends: return `{"code": 0, "data": ...}` on success, `{"code": -1, "message": ...}` on error.

## ESLint Rules of Note

- `vue/multi-word-component-names`: **off**
- `vue/require-v-for-key`: **warn**
- `vue/no-use-v-if-with-v-for`: **warn**
- `@typescript-eslint/no-this-alias`: **warn**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [intel/AI-Playground](https://github.com/intel/AI-Playground) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
