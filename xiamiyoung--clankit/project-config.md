---
trigger: always_on
description: ClanKit is a multi-LLM desktop chat application built with **Electron + Vue 3 + Vite**. It supports Anthropic (Codex), OpenRouter, and OpenAI-compatible backends. Features include agent management, MCP server integration, HTTP tools, knowledge base (RAG via local embeddings and vectra vector store), skills, and an agentic tool-use loop.
---

# AGENTS.md — ClanKit Project Guide

## Project Overview

ClanKit is a multi-LLM desktop chat application built with **Electron + Vue 3 + Vite**. It supports Anthropic (Codex), OpenRouter, and OpenAI-compatible backends. Features include agent management, MCP server integration, HTTP tools, knowledge base (RAG via local embeddings and vectra vector store), skills, and an agentic tool-use loop.

## Tech Stack

- **Runtime:** Electron 31 (main process: Node.js/CommonJS, renderer: ES modules)
- **Frontend:** Vue 3.4 (Composition API with `<script setup>`), Pinia 2 (state), Vue Router 4 (hash history)
- **Build:** Vite 5, `@vitejs/plugin-vue`
- **Styling:** Tailwind CSS 3.4 + CSS custom properties + scoped component CSS
- **Markdown:** `marked` + `highlight.js` + `DOMPurify`
- **Rich Text:** TipTap (Vue 3)
- **3D Viewer:** Babylon.js (model preview in chat)
- **Avatars:** DiceBear Avataaars
- **IDs:** `uuid` v9
- **i18n:** Custom lightweight solution (`src/i18n/`)

## i18n Multilingual Support (Prerequisite for All Development)

**All new feature development must support multiple languages.** This is not optional; it is an architectural prerequisite.

### Configuration Layer

- Language settings are stored in the `language` field of `config.json` (`'en'` | `'zh'`)
- Access through `configStore.language` (computed, automatically reacts to config changes)
- Language changes take effect immediately without restart

### Translation File Structure

```
src/i18n/
├── index.js      # Translation dictionary object (en, zh)
└── useI18n.js    # Composable: useI18n() returns { t, locale }
```

### Usage

`useI18n()` (from `src/i18n/useI18n`) returns `{ t, locale }`. Use `{{ t('key') }}` in templates.

### Key Naming Conventions

- `app.*` - Application-level (name, slogan)
- `nav.*` - Navigation items
- `common.*` - Common buttons/labels (Save, Cancel, Delete, Add, Search, etc.)
- `config.*` - Configuration pages
- `agents.*` - Agent-related
- `chats.*` - Chat-related
- `skills.*`, `knowledge.*`, `mcp.*`, `tools.*` - Feature modules

### AI Language

- `config.language` is the AI **default language**
- When creating a new agent, fields such as name and description should be generated in the current language
- Chat-level override: `chat.languageOverride` allows switching AI language within a single chat

### Migration Guide

Existing hardcoded text should be migrated one by one to `t('key')` calls. Prioritize:
1. High-frequency components (NavItem, AppButton, form labels)
2. Main pages (ConfigView, AgentsView, ChatsView)
3. Low-frequency components afterward

## Repository Structure

- `electron/` — Main process (CommonJS). `main.js` is a thin shell; all IPC handlers live in `electron/ipc/` (18 modules registered via `index.js::registerAll()`). Agent runtime in `electron/agent/`: `agentLoop.js` (core LLM loop), `systemPromptBuilder.js`, `messageConverter.js`, `toolExecutor.js`, `chunkAccumulator.js`, `dataNormalizers.js`, plus `core/` (LLM clients), `tools/`, `mcp/`, `voice/`. Shared utilities in `electron/lib/`.
- `src/` — Vue renderer (ES modules). Pinia stores in `src/stores/`, composables extracted from ChatsView in `src/composables/` (`useSendMessage`, `useChunkHandler`, `useAgentCollaboration`, `useChatTree`, `useMessageOps`, `useVoiceRecording`, etc.), components split by feature under `src/components/{chat,common,agents}/`, utilities in `src/utils/`, pages in `src/views/`.
- Root configs: `vitest.config.js`, `tailwind.config.js`, `vite.config.js`, `postcss.config.js`, `package.json`.

> Use Glob/Grep to discover current file names — this section is intentionally high-level, not an inventory.

## Commands

```bash
# Development (Vite dev server + Electron, no hot reload for Electron main process)
npm run dev

# Build Vue frontend
npm run build

# Run Electron standalone (after build)
npm run electron
```

> **No Electron hot reload:** The dev script does **not** auto-restart the Electron main process when files in `electron/` change. Vue renderer changes are still picked up by Vite HMR. Changes to `electron/main.js`, `electron/preload.js`, or any file under `electron/agent/` require a manual app restart.

## Architecture & Conventions

### Electron IPC Pattern

- Main process exposes IPC handlers via `ipcMain.handle('channel', ...)` in `electron/ipc/` modules
- Preload script (`electron/preload.js`) bridges channels to `window.electronAPI` via `contextBridge`
- Renderer accesses IPC through `window.electronAPI.methodName()`
- Namespaced IPC channels: `store:*`, `mcp:*`, `tools:*`, `agent:*`, `skills:*`, `knowledge:*`, `files:*`

### Vue Component Patterns

- **Always** use `<script setup>` Composition API (no Options API)
- Stores are Pinia setup stores (function-based `defineStore`)
- Props validated with `defineProps()`, events with `defineEmits()`
- Use `defineOptions({ inheritAttrs: false })` when forwarding `$attrs`
- Inline SVG icons defined as render-function components (see `Sidebar.vue`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [XiamiYoung/ClanKit](https://github.com/XiamiYoung/ClanKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
