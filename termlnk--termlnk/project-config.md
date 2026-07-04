---
trigger: always_on
description: **Termlnk** is a modular, plugin-based **terminal application** built with **Electron + TypeScript + React**.
---

# AGENTS.md

## Project Overview

**Termlnk** is a modular, plugin-based **terminal application** built with **Electron + TypeScript + React**.

### Tech Stack

| Category | Technology                              |
|----------|-----------------------------------------|
| Runtime | Node.js >= 22, pnpm >= 10               |
| Frontend | React 19, TanStack Router, Jotai        |
| Desktop | Electron 40, electron-vite              |
| Build | Vite 8, Turborepo, termlnk-cli          |
| Types | TypeScript 6.0 (strict)                 |
| Testing | Vitest + happy-dom                      |
| Styling | Tailwind CSS v4.2.2 (tm: prefix)        |
| DI | @wendellhu/redi 1.1.1                   |
| Reactive | RxJS (peer dependency for all packages) |
| UI | Radix UI                                |
| Themes | Base46 (base_30 + base_16), 71 presets  |
| IPC | tRPC + @janwirth/electron-trpc-link     |
| Database | Drizzle ORM + SQLite (better-sqlite3)   |

---

## Monorepo Structure

Managed with **pnpm workspaces + Turborepo**: **22 library packages + 1 app (with 2 sub-packages) + 1 internal tooling package**.

```
termlnk/
├── packages/              # 22 publishable library packages
│   ├── core/              # Foundation: DI, core services, models, plugin system
│   ├── themes/            # Theme definitions (Base46)
│   ├── themes-ui/         # Theme UI components (picker, editor)
│   ├── design/            # UI component library (Radix UI + Tailwind, 39 components)
│   ├── ui/                # Application UI layer (business components, 15+ services)
│   ├── network/           # HTTP client with interceptor pipeline + WebSocket
│   ├── database/          # Database layer (Drizzle ORM + SQLite)
│   ├── electron/          # Electron common interfaces (IWindowManagerService, IUpdaterService)
│   ├── electron-main/     # Electron main process (window management, auto-update, file I/O)
│   ├── electron-renderer/ # Electron renderer process (tRPC client, header UI, update UI)
│   ├── terminal/          # Terminal core (config, host models, CSI/DCS/OSC parsers, PTY, Shell Integration)
│   ├── terminal-ui/       # Terminal UI (sessions, workspace splits, search, drag-drop, local terminal)
│   ├── rpc/               # RPC base types (Observable utilities, SSH/SFTP/PTY models)
│   ├── rpc-server/        # RPC server (tRPC 14 routers, SSH, SFTP, PTY, AI, Skill)
│   ├── rpc-client/        # RPC client (12 facade services)
│   ├── agent/             # Agent shared contracts (AI, Skill, MCP interfaces, types, DI identifiers)
│   ├── agent-core/        # AI Agent main process (reasoning engine, MCP server/client, Skill management)
│   ├── agent-ui/          # AI Agent renderer UI (chat panel, model selector, settings)
│   ├── extension/         # Extension system (loader, manifest, contribution points, extension API)
│   ├── extension-ui/      # Extension management UI (browse, enable/disable)
│   ├── settings-ui/       # Settings panel UI (9 tabs: appearance, terminal, network, AI, MCP, Skill...)
│   └── sftp-ui/           # SFTP file browser UI (dual-pane layout, file transfer)
├── apps/
│   └── desktop/           # Desktop application (not published)
│       ├── main/          # Main process (6 plugins)
│       └── renderer/      # Renderer process (12 plugins, React app)
├── internal/
│   └── shared/            # Build tools and configs (termlnk-cli, ESLint presets, Vite builder)
└── docs/
```

### Package Dependency Layers

```
Foundation
  @termlnk/core → @termlnk/themes (peer)

Data
  @termlnk/database → core

Communication
  @termlnk/network → core
  @termlnk/rpc → core
  @termlnk/rpc-server → rpc, database, terminal, agent
  @termlnk/rpc-client → rpc, terminal, agent

Electron
  @termlnk/electron → core
  @termlnk/electron-main → electron, rpc-server, terminal, agent
  @termlnk/electron-renderer → electron, rpc-client, ui

UI
  @termlnk/design → (no deps, peer: react)
  @termlnk/ui → core, design, themes
  @termlnk/themes-ui → core, design, themes, ui

Terminal
  @termlnk/terminal → core
  @termlnk/terminal-ui → core, design, rpc, rpc-client, terminal, themes, ui

Agent
  @termlnk/agent → core (pure contracts)
  @termlnk/agent-core → core, agent, database, rpc, terminal
  @termlnk/agent-ui → core, agent, design, rpc-client, ui

Extension
  @termlnk/extension → core, rpc-client
  @termlnk/extension-ui → core, design, extension, rpc-client, ui

Settings / SFTP
  @termlnk/settings-ui → core, design, rpc-client, terminal, themes, themes-ui, ui, agent-ui, agent
  @termlnk/sftp-ui → core, design, rpc, rpc-client, ui
```

**All packages share peer dependency**: `rxjs >= 7.0.0`

Each package has its own `CLAUDE.md` with detailed architecture and API documentation.

---

## Process Architecture

Termlnk uses Electron dual-process architecture with tRPC over IPC:

```
┌──────────────────────────────────────────────────┐
│  Main Process (Node.js)                          │
│  Plugins: Database, RPC, AgentCore,              │
│           RPCServer, Electron, ElectronMain       │
│  Services: SSH, SFTP, PTY, AI Agent, MCP,        │
│  FileTransfer, Extension, Database, Updater      │
└──────────────────┬───────────────────────────────┘

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [termlnk/termlnk](https://github.com/termlnk/termlnk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
