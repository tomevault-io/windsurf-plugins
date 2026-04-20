---
trigger: always_on
description: > This file provides context and instructions for AI coding agents.
---

# AGENTS.md — Motrix Next

> This file provides context and instructions for AI coding agents.
> For human contributors, see [README.md](README.md) and [CONTRIBUTING.md](docs/CONTRIBUTING.md).

> [!IMPORTANT]
> **All changes must meet industrial-grade quality.** Enforce DRY (extract composables/utilities over duplication), strict TypeScript (no `any`, justify every `as` cast), structured error handling, and full verification (`vue-tsc` + tests pass) before completion.

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
│   ├── configKeys.ts           # Config key lists (userKeys, systemKeys, needRestartKeys)
│   ├── logger.ts               # Structured logging (console + webview bridge)
│   ├── timing.ts               # Timing constants (polling intervals, debounce delays)
│   ├── guards.ts               # Type guard utilities
│   ├── locales/                # 26 locale directories (see Section D)
│   └── utils/
│       ├── configMigration.ts  # Config schema migration engine (see Section C′)
│       ├── config.ts           # Config key-value transform utilities
│       ├── tracker.ts          # BT tracker fetching with proxy support
│       ├── geoip.ts            # GeoIP peer lookup (country code → flag)
│       ├── fileCategory.ts     # File type classification by extension
│       ├── autoArchive.ts      # Auto-archive completed tasks
│       ├── format.ts           # Number/date/speed formatting (bytesToSize, localeDateTimeFormat)
│       ├── task.ts             # Task status helpers (checkTaskIsBT, getTaskDisplayName)
│       ├── peer.ts             # Peer ID parsing and client identification
│       └── semver.ts           # Semantic version comparison for update channel
├── stores/                     # Pinia stores (app.ts, preference.ts, history.ts, task/)
├── views/                      # Page-level route views
└── main.ts                     # App entry, auto-update check

src-tauri/
├── src/
│   ├── lib.rs                  # Tauri builder, plugin registration, invoke_handler
│   ├── main.rs                 # Tauri entry point
│   ├── aria2/                  # Native Rust aria2 JSON-RPC client
│   │   ├── mod.rs              # Module re-exports
│   │   ├── client.rs           # WebSocket JSON-RPC client (connect, call, subscribe)
│   │   └── types.rs            # Aria2 response types (Aria2Task, Aria2File, Aria2BtInfo, etc.)
│   ├── commands/
│   │   ├── mod.rs              # Command module re-exports
│   │   ├── aria2.rs            # aria2 JSON-RPC forwarding (tell_active, global_stat, etc.)
│   │   ├── config.rs           # Config CRUD, session, factory reset commands
│   │   ├── engine.rs           # Engine start/stop/restart commands
│   │   ├── fs.rs               # File system ops, diagnostics, platform code
│   │   ├── geoip.rs            # GeoIP database loading and peer IP lookup
│   │   ├── history.rs          # History DB read/write commands
│   │   ├── net.rs              # Network utility commands
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
│   │   ├── args.rs             # aria2 command-line argument builder
│   │   ├── cleanup.rs          # Engine cleanup utilities
│   │   └── state.rs            # Engine state management
│   ├── services/
│   │   ├── mod.rs              # Runtime services orchestration (on_engine_ready)
│   │   ├── config.rs           # RuntimeConfig cache (refreshed per engine cycle)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AnInsomniacy/motrix-next](https://github.com/AnInsomniacy/motrix-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
