---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UI-TARS Desktop is a monorepo containing two independent projects:

1. **UI-TARS Desktop** (`apps/ui-tars/`) — An Electron app that acts as a GUI agent, using UI-TARS vision-language models to control the user's computer via natural language.
2. **Agent TARS** (`multimodal/`) — A separate monorepo for a multimodal AI agent stack with its own pnpm workspace, CLI, and web UI.

These two projects share some infrastructure packages but have separate workspaces and build systems.

## Common Commands

### Root monorepo (UI-TARS Desktop)
```bash
pnpm install                          # Install dependencies
pnpm run dev:ui-tars                  # Start Electron app in dev mode
pnpm run test                         # Run unit tests (vitest)
pnpm run coverage                     # Run tests with coverage
pnpm run lint                         # ESLint with auto-fix
pnpm run format                       # Prettier formatting
```

### UI-TARS Desktop app (`apps/ui-tars/`)
```bash
pnpm run dev                          # Electron dev with HMR
pnpm run dev:w                        # Dev with main process reload
pnpm run typecheck                    # TypeScript checking (node + web configs)
pnpm run build                        # Full production build with installers
pnpm run build:dist                   # Build dist files only (no installer)
pnpm run build:e2e && pnpm run test:e2e  # E2E tests (Playwright, requires build first)
pnpm run debug                        # Dev with remote debugging on port 9222
```

### Running a single test
```bash
pnpm vitest run path/to/test.ts                # Single file
pnpm vitest run --grep "test name pattern"     # By name pattern
```

Tests use vitest workspaces (`vitest.workspace.mts` references `src/*` and `packages/*`). Coverage uses Istanbul provider with text, json, html, and lcov reporters.

### Agent TARS (`multimodal/`)
```bash
cd multimodal
pnpm install && pnpm run build        # Bootstrap (or: pnpm run bootstrap)
pnpm run dev                          # Dev mode (uses ptk)
pnpm run test                         # Run tests (vitest)
```

## Architecture

### Monorepo Layout
- **Root pnpm workspace** manages `apps/*`, `packages/**` — everything for UI-TARS Desktop
- **`multimodal/`** is a separate pnpm workspace with its own `pnpm-lock.yaml` — manages `tarko/*`, `agent-tars/*`, `gui-agent/*`, `omni-tars/*`, `benchmark/*`
- **Turbo** (`turbo.json`) orchestrates build tasks at the root level only

### Electron App (`apps/ui-tars/`)
- Built with **electron-vite** (`electron.vite.config.ts`) which configures three separate bundles:
  - `src/main/main.ts` → main process (uses bytecodePlugin for protection)
  - `src/preload/index.ts` → preload bridge
  - `src/renderer/` → React UI (Vite + React + Tailwind CSS 4)
- Packaged with **Electron Forge** (`forge.config.ts`) — DMG for macOS, Squirrel for Windows
- Main process structure (`src/main/`):
  - `ipcRoutes/` — IPC handlers organized by domain: `agent`, `browser`, `permission`, `screen`, `setting`, `window`, `remoteResource`
  - `agent/` — Agent logic including operator setup and prompts
  - `services/`, `store/`, `window/`, `remote/` — Business logic, state, window management
- Renderer (`src/renderer/`) — React + Zustand for state, standard pages/components/hooks layout

### Core Agent Loop (the key data flow)
The `GUIAgent` class in `packages/ui-tars/sdk` orchestrates the main loop:
1. **Screenshot** — Operator captures the screen (`Operator.screenshot()`)
2. **Model inference** — Screenshot sent to UI-TARS VLM via `UITarsModel` (OpenAI-compatible API)
3. **Action parsing** — Model output parsed by `@ui-tars/action-parser` into structured actions
4. **Execution** — Actions executed by the platform-specific `Operator` (nut-js for desktop, browser operator, ADB for Android)
5. **Loop** — Repeat until task complete or max iterations reached

The `Operator` is an abstract base class (`packages/ui-tars/sdk/src/types.ts`) that each platform implements. Available operators live in `packages/ui-tars/operators/` (nut-js, browser-operator, adb, browserbase).

### Key Packages (`packages/`)
- `packages/ui-tars/sdk` — Core SDK: `GUIAgent`, `UITarsModel`, `Operator` base class
- `packages/ui-tars/action-parser` — Parses model text output into executable action structs
- `packages/ui-tars/operators/` — Platform operators (nut-js desktop, browser, adb, browserbase)
- `packages/ui-tars/electron-ipc` — Type-safe IPC between Electron main/renderer
- `packages/ui-tars/utio` — UI automation I/O utilities
- `packages/ui-tars/shared` — Shared types (`StatusEnum`, `GUIAgentData`, `Message`) and constants
- `packages/agent-infra/` — Agent infrastructure: browser automation, MCP client/servers (browser, commands, filesystem, search), logging
- `packages/common/configs` — Shared ESLint and TypeScript configs
- `packages/common/electron-build` — Electron build configuration and hooks

### Agent TARS (`multimodal/`)
- `tarko/` — Low-level agent framework (~24 packages): event-stream driven agent, LLM integration, MCP agent, UI

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yangzhuxinyzx/auto-openclaw](https://github.com/yangzhuxinyzx/auto-openclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
