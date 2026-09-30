---
trigger: always_on
description: > This file provides context and instructions for AI coding agents.
---

# AGENTS.md — Rayburst

> This file provides context and instructions for AI coding agents.
> For human contributors, see [README.md](README.md) and [CONTRIBUTING.md](docs/CONTRIBUTING.md).

> [!IMPORTANT]
> **All changes must meet industrial-grade quality.** Enforce DRY (extract composables/utilities over duplication), strict TypeScript (no `any`, justify every `as` cast), structured error handling, and verification proportional to the changed behavior.

---

## A. Project Architecture

| Layer               | Stack                                                              |
| ------------------- | ------------------------------------------------------------------ |
| **Frontend**        | Vue 3 Composition API + Pinia + Naive UI + TypeScript              |
| **Backend**         | Rust (Tauri 2) + aria2 sidecar                                     |
| **Build**           | Vite (frontend) + Cargo (backend)                                  |
| **Package Manager** | pnpm (version pinned via `packageManager` field in `package.json`) |
| **Testing**         | Vitest (frontend), cargo test (backend)                            |

### Key File Paths

```
src/
├── api/                        # Aria2 JSON-RPC client (frontend wrapper)
├── components/preference/      # Settings UI (Basic.vue, Advanced.vue, UpdateDialog.vue)
├── composables/                # Vue composables — business logic extracted from components
├── layouts/                    # Page-level layouts (MainLayout.vue)
├── shared/
│   ├── types.ts                # All TypeScript interfaces (AppConfig, TauriUpdate, etc.)
│   ├── constants.ts            # DEFAULT_APP_CONFIG, proxy scopes, tracker URLs, timing constants
│   ├── configKeys.ts           # needRestartKeys + re-exports of aria2Options.json lists
│   ├── aria2Options.json       # SINGLE SOURCE for engine option lists (TS + Rust both consume)
│   ├── logger.ts               # Structured logging (console + webview bridge)
│   ├── timing.ts               # Timing constants (polling intervals, debounce delays)
│   ├── guards.ts               # Type guard utilities
│   ├── locales/                # 27 locale directories (see Section D)
│   └── utils/
│       ├── configHydration.ts  # Current config defaults, nested merge, and validation
│       ├── config.ts           # Config key-value transform utilities
│       ├── tracker.ts          # BT tracker fetching with proxy support
│       ├── geoip.ts            # GeoIP peer lookup (country code → flag)
│       ├── fileCategory.ts     # Category editing and native preview requests
│       ├── format.ts           # Number/date/speed formatting (bytesToSize, localeDateTimeFormat)
│       ├── task.ts             # Task status helpers (checkTaskIsBT, getTaskName)
│       ├── peer.ts             # Peer ID parsing and client identification
│       └── proxy.ts            # Proxy policy, URL building/validation, engine option assembly
├── stores/                     # Pinia stores (app.ts, preference.ts, history.ts, task/)
├── views/                      # Page-level route views
└── main.ts                     # App entry, auto-update check

src-tauri/
├── src/
│   ├── lib.rs                  # Tauri builder, plugin registration, invoke_handler
│   ├── main.rs                 # Tauri entry point
│   ├── aria2/                  # Native Rust aria2 JSON-RPC client
│   │   ├── mod.rs              # Module re-exports
│   │   ├── rpc.rs              # HTTP JSON-RPC transport, authentication and structured errors
│   │   └── types.rs            # Aria2 response types (Aria2Task, Aria2File, Aria2BtInfo, etc.)
│   ├── commands/
│   │   ├── mod.rs              # Command module re-exports
│   │   ├── aria2.rs            # aria2 JSON-RPC forwarding (tell_active, global_stat, etc.)
│   │   ├── config.rs           # Config CRUD, session, factory reset commands
│   │   ├── engine.rs           # Engine start/stop/restart commands
│   │   ├── fs.rs               # File system ops, diagnostics, platform code
│   │   ├── geoip.rs            # GeoIP database loading and peer IP lookup
│   │   ├── history.rs          # History DB read/write commands
│   │   ├── http_api.rs         # Local extension HTTP API auth and status commands
│   │   ├── remote_file.rs      # Remote torrent download and metainfo inspection
│   │   ├── notification.rs     # Native notification permission and test commands
│   │   ├── power.rs            # System power action commands
│   │   ├── protocol.rs         # Default protocol handler detection and registration
│   │   ├── proxy.rs            # System proxy detection (PAC, WPAD, env)
│   │   ├── runtime_config.rs   # RuntimeConfig refresh command
│   │   ├── tracker.rs          # Tracker probing and protocol classification
│   │   ├── ui.rs               # Tray, menu, dock, progress bar commands
│   │   ├── updater.rs          # check_for_update, download_update, apply_update, cancel_update
│   │   └── upnp.rs             # UPnP port mapping commands
│   ├── engine/
│   │   ├── mod.rs              # Module re-exports
│   │   ├── lifecycle.rs        # aria2 sidecar start/stop/restart
│   │   ├── config.rs           # Managed runtime aria2.conf generation
│   │   ├── cleanup.rs          # Engine cleanup utilities

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AnInsomniacy/rayburst](https://github.com/AnInsomniacy/rayburst) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
