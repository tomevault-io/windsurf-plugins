---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# ahaCode AI Coding Guidelines

You are an expert Senior Frontend Developer specializing in Electron, Vue 3, and TypeScript.
Follow these rules strictly when generating code for ahaCode.

## 1. Core Stack

- **Framework:** Vue 3 (Composition API, `<script setup lang="ts">`)
- **Styling:** TailwindCSS v4 (`@tailwindcss/vite`), `tailwind-merge`, `cva`
- **UI:** Custom components (`src/renderer/components/ui`), Shadcn (based on `reka-ui`), `lucide-vue-next` icons
- **State:** Vue Composables (No Vuex/Pinia)
- **Backend:** Electron (Main), `better-sqlite3` (DB), Elysia.js (API)
- **Utilities:** `@vueuse/core`, `vue-sonner` (Notifications)

**Path Aliases:**
- `@` → `src/renderer` (renderer code)
- `~` → `src` (any source code, including main/shared)

**Dual TypeScript Configs:**
- `tsconfig.json` — Renderer process (Vue/Vite, ESNext modules)
- `tsconfig.main.json` — Main process + shared (CommonJS, compiled to `build/`)

**Node Version:** Pinned via Volta (`node 24.14.1`). Use `pnpm` (>=10) as the package manager.

## 2. Philosophy

**YAGNI — simplicity above all.** Do not overcomplicate code for hypothetical future scenarios. The minimum viable implementation is the correct implementation. Three similar lines of code are better than a premature abstraction.

Signs of overengineering:
- A function guards against a case that will never happen
- A factory used in exactly one place that doesn't encapsulate state
- Abstraction for its own sake (a wrapper around a single line of code)
- Constants or patterns invented in advance without a real need

## 3. Architecture & Communication

**Strict Separation of Concerns:**

| Layer        | Process  | Access                                      | Communication                             |
|:-------------|:---------|:--------------------------------------------|:------------------------------------------|
| **Renderer** | Frontend | **NO** Node.js/DB access. Only via API/IPC. | Calls API (`api.*`) or IPC (`ipc.invoke`) |
| **API**      | Main     | Full DB/System access.                      | Receives requests from Renderer           |
| **Main**     | Backend  | Full System access.                         | Handles IPC & Lifecycle                   |

**Data Flow:** Renderer → REST API (Elysia) → Service/DB Layer → Response

## 4. File Naming

| Type | Convention | Example |
|------|------------|---------|
| Vue components | PascalCase | `Folders.vue`, `CreateDialog.vue` |
| TypeScript files | camelCase | `useSnippets.ts`, `errorMessage.ts` |

Composables get a `use` prefix. The file name matches the exported function name: `useSnippets.ts` → `export function useSnippets()`.

## 5. Critical Rules & Conventions

### A. Imports (STRICT)

**❌ DO NOT IMPORT:**

- Vue core (`ref`, `computed`, `watch`, `onMounted`) → *Auto-imported*
- Project components (`src/renderer/components/`) → *Auto-imported* (e.g., `<SidebarFolders />` for `components/sidebar/Folders.vue`)

**✅ ALWAYS IMPORT MANUALLY:**

- Shadcn UI: `import { Button } from '@/components/ui/shadcn/button'` or `import * as Select from '@/components/ui/shadcn/select'`
- Composables: `import { useApp } from '@/composables'`
- Utils: `import { cn } from '@/utils'`
- VueUse: `import { useClipboard } from '@vueuse/core'`
- Electron IPC/Store: `import { ipc, store } from '@/electron'`

### B. State & Settings

- **Global State:** Composables in `@/composables` (e.g., `useApp`, `useSnippets`) maintain shared state by defining reactive variables **outside** the exported function (module level). This ensures all components access the same state. **No Pinia/Vuex.**
- **Persistent Settings:** Use `store` from '@/electron'.
  - `store.app`: UI state (sizes, visibility)
  - `store.preferences`: User prefs (theme, language)

### C. Database & API

- **Renderer:** **NEVER** import `better-sqlite3`. Use `import { api } from '~/renderer/services/api'`
- **New Endpoints:**
  1. Define DTO in `src/main/api/dto/`
  2. Add route in `src/main/api/routes/`
  3. **Run `pnpm api:generate`** to update client.
- **Generated API Client:** `src/renderer/services/api/generated/` — auto-generated, never edit manually. Excluded from ESLint.

**Storage Engine Architecture:**

The app supports two storage backends, selectable by the user:

| Engine | Provider | Location |
|--------|----------|----------|
| `sqlite` | `better-sqlite3` via `src/main/db/` | Single `.db` file |
| `markdown` | File-based in `src/main/storage/providers/markdown/` | Vault directory with `.md` files, YAML frontmatter, `state.json` |

Both implement the `StorageProvider` / `NotesStorageProvider` interfaces from `src/main/storage/contracts.ts`. The markdown provider includes a file watcher (`watcher.ts`) for external sync and migration support (`migrations.ts`).

API routes are storage-engine agnostic — they call the active provider through `src/main/storage/index.ts`.

### D. System & IPC

- **File System/System Ops:** Use `ipc.invoke('channel:action', data)`.
- **Channels:** `fs:*`, `system:*`, `db:*`, `main-menu:*`, `prettier:*`, `spaces:*`, `theme:*`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sinksmell/ahaCode](https://github.com/sinksmell/ahaCode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
