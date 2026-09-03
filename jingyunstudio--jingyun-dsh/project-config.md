---
trigger: always_on
description: **jingyun-dsh** (`@jingyun-ai/jingyun-dsh` / `Jingyun.Studio`) is a desktop application and enterprise plugin distribution for the **DeepSeek Harness (DSH)** AI Agent orchestration platform. It provides custom branding, Cordis-based agent management hooks, cloud/local tenant synchronization, REST APIs, and a standalone desktop distribution via a Rust Tauri v2 shell.
---

# Repository Guidelines

## Project Overview
**jingyun-dsh** (`@jingyun-ai/jingyun-dsh` / `Jingyun.Studio`) is a desktop application and enterprise plugin distribution for the **DeepSeek Harness (DSH)** AI Agent orchestration platform. It provides custom branding, Cordis-based agent management hooks, cloud/local tenant synchronization, REST APIs, and a standalone desktop distribution via a Rust Tauri v2 shell.

---

## Architecture & Data Flow

```
+-------------------------------------------------------------------------------+
|                           Tauri v2 Desktop Shell (Rust)                       |
|   - Spawns Node.js Sidecar under Win32 Job Object (KillOnClose lifecycle)     |
|   - Custom Window Controls (IPC: min/max/close/drag), Tray Icon, Splash       |
+-------------------------------------------------------------------------------+
                                      |
                                      v (Loads via HTTP / Localhost)
+-------------------------------------------------------------------------------+
|                      DSH Core Server Runtime (Node.js ESM)                    |
|   - Cordis IoC Container (@deepseek-ai/cordis)                                |
|   - Jingyun Backend Plugin (@jingyun-ai/jingyun-dsh)                          |
|     * REST API Routes: /api/jingyun/* (tenant, agents, config, marketplace)   |
|     * Agent Loader: Hooks systemPrompt to inject active personas / workspace  |
|     * Skills Manager: Synchronizes builtin skills to $DSH_HOME/skills         |
|     * Settings/Schema: Schemastery configuration management                   |
+-------------------------------------------------------------------------------+
                                      |
                                      v (Slot Injections & HTTP Requests)
+-------------------------------------------------------------------------------+
|                    DSH Web Client / React 19 Frontend                         |
|   - Custom Module Loader (__ModuleLoader__) injects lib/client.js             |
|   - Extension Slots: sidebar.brand.*, conversation.hero.*, sidebar.footer.*   |
|   - Dynamic Branding: MutationObserver title/favicon & CSS variable overrides |
+-------------------------------------------------------------------------------+
```

### Data Flow
1. **Startup**: Tauri boots -> extracts portable vendor packages if needed -> spawns Node.js DSH backend sidecar -> attaches to Windows Job Object for clean termination -> opens Webview.
2. **Plugin Initialization**: DSH loads `@jingyun-ai/jingyun-dsh` via Cordis service container -> registers REST endpoints (`/api/jingyun/*`), synchronizes builtin skills to `$DSH_HOME/skills`, and registers `systemPrompt` transform hooks.
3. **Frontend Slot Injection**: Client frontend script (`lib/client.js`) registers React components into DSH UI extension slots (branding banners, agent switchers, settings tabs).
4. **Agent Dynamic Injection**: On conversation creation, `agent-loader.ts` intercepts `systemPrompt` via Cordis to insert custom agent system prompts and workspace rules.

---

## Key Directories

| Directory | Purpose |
|---|---|
| `packages/jingyun-dsh/` | Core plugin package containing both backend (Cordis) and frontend (React client) code. |
| `packages/jingyun-dsh/src/` | TypeScript source for the plugin. |
| `packages/jingyun-dsh/src/agent/` | Dynamic agent loaders, active agent managers, and template generators. |
| `packages/jingyun-dsh/src/client/` | React 19 UI components injected into DSH extension slots, styles, and DOM observers. |
| `packages/jingyun-dsh/src/routes/` | REST API routes mounted under `/api/jingyun/*`. |
| `packages/jingyun-dsh/src/common/` | Shared utilities: path resolution, HTTP helpers, logger, and tenant discovery. |
| `src-tauri/` | Rust Tauri v2 desktop application shell, window management, and sidecar process controller. |
| `src-tauri/resources/builtin-skills/` | Builtin skills (e.g. `agent-manager`, `skill-creator`) synced to user environment. |
| `scripts/` | Build, packaging (`pack_gui.py`), vendor preparation, and environment setup scripts. |

---

## Development Commands

### Package & Dependency Management
- **Install dependencies**: `pnpm install`
- **Typecheck codebase**: `pnpm typecheck` (`tsc -b --noEmit`)
- **Lint code**: `pnpm lint` (`oxlint --fix`)
- **Format code**: `pnpm fmt` (`oxfmt --write`)

### Plugin Development
- **Run Plugin in Watch Mode**: `pnpm watch` (or `pnpm -C packages/jingyun-dsh dev`)
- **Build Plugin**: `pnpm build` (runs `prepare:vendor` + `tsdown` build in packages)
- **Start Dev Host**: `pnpm dev` (runs watcher + DSH dev server concurrently)
- **Start DSH Standalone**: `pnpm start`

### Desktop (Tauri) & Packaging
- **Run Tauri Dev**: `pnpm tauri:dev` (`tauri dev`)
- **Build Tauri Release**: `pnpm tauri:build` (builds plugin, prepares vendor bundles, and outputs NSIS installer)
- **Build Vendor Dependencies Zip**: `pnpm build:deps`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jingyunstudio/jingyun-dsh](https://github.com/jingyunstudio/jingyun-dsh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
