---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

SkillsFan is a cross-platform desktop application that wraps Codex in a visual interface. Built with Electron + React + TypeScript using `electron-vite`, it provides a GUI alternative to the CLI while maintaining full agent capabilities.

## Development Commands

```bash
npm run dev              # Start dev server (uses ~/.skillsfan-dev for data)
npm run build            # Build application
npm run test             # Run all tests (check + unit)
npm run test:unit        # Unit tests only (vitest)
npm run test:unit:watch  # Unit tests in watch mode
npm run test:e2e         # E2E tests (Playwright)
npm run i18n             # Extract and translate i18n strings (run before committing new UI text)
```

Run a single unit test:
```bash
vitest run tests/unit/services/space.test.ts --config tests/vitest.config.ts
```

Run a single E2E test project:
```bash
playwright test --config tests/playwright.config.ts --project=smoke
```

Region-specific builds:
```bash
npm run build:mac:cn         # macOS for China region
npm run build:mac:overseas   # macOS for overseas region
npm run build:win:cn         # Windows for China region
npm run build:win:overseas   # Windows for overseas region
```

## Architecture

### Electron Process Model

```
┌─────────────────────────────────────────────────────────────────┐
│                          Main Process                           │
│  ┌─────────────┐    ┌─────────────────────┐                     │
│  │  Bootstrap  │───►│     Services        │                     │
│  │  (phased)   │    │  (agent, config,    │                     │
│  └─────────────┘    │   space, remote...) │                     │
│         │           └─────────────────────┘                     │
│         ▼                      │                                │
│  ┌─────────────┐               │ IPC                            │
│  │ IPC Handlers│◄──────────────┘                                │
│  └─────────────┘                                                │
└─────────────────┬───────────────────────────────────────────────┘
                  │ contextBridge (preload)
┌─────────────────▼───────────────────────────────────────────────┐
│                      Renderer Process                           │
│  ┌──────────────┐   ┌─────────────┐   ┌────────────────────┐    │
│  │ React Pages  │──►│   Stores    │──►│ API (IPC/HTTP)     │    │
│  │  (UI)        │   │  (Zustand)  │   │ Unified interface  │    │
│  └──────────────┘   └─────────────┘   └────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

### Key Directories

- `src/main/` - Electron main process
  - `services/` - Business logic (agent, config, space, conversation, ai-browser, etc.)
  - `ipc/` - IPC handlers mapping to services
  - `bootstrap/` - Phased initialization (Essential vs Extended services)
  - `http/` - Remote access HTTP/WebSocket server (Express + ws)
- `src/preload/` - Preload scripts exposing IPC to renderer
- `src/renderer/` - React frontend
  - `pages/` - Page components (HomePage, SpacePage, SettingsPage)
  - `stores/` - Zustand state management
  - `api/` - Unified API adapter (works for both IPC and HTTP modes)
  - `components/` - UI components
  - `i18n/` - Internationalization (locales: en, zh-CN, zh-TW, ja, es, fr, de)
- `src/shared/` - Types and interfaces shared between main and renderer
- `patches/` - patch-package patches (notably for `@anthropic-ai/Codex-agent-sdk`)

### Path Aliases (tsconfig)

- `@/*` → `src/renderer/*`
- `@main/*` → `src/main/*`
- `@shared` / `@shared/*` → `src/shared/*`

### Bootstrap Phases

The app uses two-phase initialization to optimize startup time (see `src/main/bootstrap/`):

1. **Essential** - Services required for first screen render (<500ms target): Config, Space, Conversation, Agent, Artifact, System, Updater, Auth
2. **Extended** - All other services, loaded after window is visible: Onboarding, Remote, Browser, AI Browser, Overlay, Search, etc.

When adding new services, default to Extended unless the feature is needed immediately on app open. Main sends `bootstrap:extended-ready` to renderer when extended services finish loading.

### Adding New IPC Channels

When adding a new IPC event, update these 3 files:
1. `src/preload/index.ts` - Expose to `window.skillsfan`
2. `src/renderer/api/transport.ts` - Add to `methodMap` (for event listeners)
3. `src/renderer/api/index.ts` - Export unified API method

### Dual Transport Architecture

The renderer API (`src/renderer/api/index.ts`) provides a unified interface that works in:
- **Electron mode** - Uses IPC via `window.skillsfan`
- **Remote mode** - Uses HTTP/WebSocket for remote access from browsers

### IPC Response Convention

All IPC handlers return `{ success: boolean, data?: T, error?: string }`.

### Event Naming

IPC events follow the format `{service}:{event}` (e.g., `agent:message`, `browser:state-change`).

### Agent Service

The agent integration lives in `src/main/services/agent/` with a modular split:
- `session-manager.ts` - Session lifecycle (create, warm, close); sessions are cached per conversation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZhuoyuBai/SkillsFanDesktop](https://github.com/ZhuoyuBai/SkillsFanDesktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
