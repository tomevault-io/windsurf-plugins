---
trigger: always_on
description: D2RMM is an **Electron + React + TypeScript** desktop app — a mod manager for Diablo II: Resurrected. It uses `electron-react-boilerplate` as its structural foundation with webpack for bundling.
---

# D2RMM — Claude Code Guide

D2RMM is an **Electron + React + TypeScript** desktop app — a mod manager for Diablo II: Resurrected. It uses `electron-react-boilerplate` as its structural foundation with webpack for bundling.

---

## Development Commands

```bash
yarn start          # Dev mode (launches webpack-dev-server + Electron)
yarn build          # Production build (main + renderer)
yarn package        # Full release build (electron-builder → zip/dmg)
yarn test           # Jest unit tests
yarn lint           # ESLint
yarn typecheck      # tsc --noEmit
yarn docs           # Generate TypeDoc site
yarn build:updater  # Build auto-updater binaries (requires pkg)
yarn build:casclib  # Build CascLib native module (requires node-gyp)
```

> CI uses `npm` (not yarn), despite `yarn.lock`. Local dev uses `yarn`. Node v22 required.

> **Local machine config** (shell, paths, tools): see @.claude/local.md

---

## Architecture Overview

Three distinct processes communicate via a type-safe IPC/RPC system:

```
Renderer (React UI)
    ↕ IPC (consumeAPI / provideAPI)
Main Process (Electron)
    ↕ Worker thread messages
Worker Thread (heavy lifting: mod install, Nexus API, file I/O)
```

All inter-process calls are typed through **bridge interfaces** in [src/bridge/](src/bridge/). See [.claude/architecture.md](.claude/architecture.md) for the full IPC pattern.

---

## Directory Structure

```
src/
├── bridge/          # Type definitions shared across all processes (IPC contracts)
├── main/            # Electron main process
│   └── worker/      # Worker thread implementations
├── renderer/        # React frontend
│   └── react/
│       ├── context/ # React Context providers + custom hooks (business logic)
│       ├── modlist/ # Mod list UI components
│       ├── ed2r/    # D2 save file editor
│       ├── mmsettings/  # App settings UI
│       └── hooks/   # Generic React hooks
├── shared/          # Code used by both main and renderer
├── types/           # Ambient TypeScript type definitions
└── updater/         # Auto-updater binary source
```

Key config locations:

- `.erb/configs/` — webpack configs (main prod, renderer dev/prod, base, paths)
- `.erb/scripts/` — build scripts (electron-rebuild, prepackage, clean, etc.)
- `.github/workflows/main.yml` — CI/CD (5 jobs: check-version, build-windows, build-macos, release, docs)
- `release/app/package.json` — runtime dependencies (what ships in the app)

---

## Key Files to Know

| What                           | Where                                                                                      |
| ------------------------------ | ------------------------------------------------------------------------------------------ |
| Root app entry (Electron main) | [src/main/main.ts](src/main/main.ts)                                                       |
| IPC routing                    | [src/main/IPC.ts](src/main/IPC.ts)                                                         |
| Core mod installation logic    | [src/main/worker/BridgeAPI.ts](src/main/worker/BridgeAPI.ts)                               |
| Nexus Mods integration         | [src/main/worker/ModUpdaterAPI.ts](src/main/worker/ModUpdaterAPI.ts)                       |
| nxm:// protocol handler        | [src/main/NxmProtocolAPI.ts](src/main/NxmProtocolAPI.ts)                                   |
| Root React component           | [src/renderer/react/App.tsx](src/renderer/react/App.tsx)                                   |
| Mod list UI                    | [src/renderer/react/modlist/ModList.tsx](src/renderer/react/modlist/ModList.tsx)           |
| Bridge type: core API          | [src/bridge/BridgeAPI.d.ts](src/bridge/BridgeAPI.d.ts)                                     |
| Bridge type: Nexus API         | [src/bridge/ModUpdaterAPI.d.ts](src/bridge/ModUpdaterAPI.d.ts)                             |
| Bridge type: Nexus data types  | [src/bridge/NexusModsAPI.d.ts](src/bridge/NexusModsAPI.d.ts)                               |
| Webpack renderer dev config    | [.erb/configs/webpack.config.renderer.dev.ts](.erb/configs/webpack.config.renderer.dev.ts) |

---

## Adding a New Feature (Checklist)

### New IPC-backed feature (most common):

1. **Define the type** in the appropriate `src/bridge/*.d.ts` file
2. **Implement** in `src/main/worker/` (or `src/main/` for non-worker APIs)
3. **Expose** via `provideAPI()` in the worker/main entry
4. **Consume** in React via `consumeAPI()` — wrap in a custom hook under `src/renderer/react/context/hooks/`
5. **Wire the hook** into a Context provider if it needs to be globally accessible, or call it directly from a component

### New UI component:

- Place in `src/renderer/react/modlist/` (mod actions/menus) or appropriate subdirectory
- Use MUI components (`@mui/material`)
- Dialog management: use `useDialogContext()` from `DialogContext`
- Toast notifications: use `useToastContext()`
- Nexus API key access: use `useNexusModsContext()`
- Mod list access: use `useModsContext()`

### New dialog:

- Create a `MyFeatureDialog.tsx` component
- Use `useDialogContext().showDialog(...)` to open it

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [olegbl/d2rmm](https://github.com/olegbl/d2rmm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
