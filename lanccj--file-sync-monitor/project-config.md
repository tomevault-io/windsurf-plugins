---
trigger: always_on
description: **Updated:** 2026-05-23
---

# PROJECT KNOWLEDGE BASE

**Updated:** 2026-05-23
**Stack:** Tauri 2, Rust 2021, Vanilla HTML/CSS/JS, SQLite, macOS + Windows

## OVERVIEW

FileSyncMonitor is now a cross-platform desktop app with a two-sided architecture:

- `desktop-portal/`: the Web UI frontend bundled by Tauri.
- `sync-kernel/`: the Rust/Tauri backend that owns file watching, SQLite persistence, tray integration, IMA sync, login capture, and OS-level commands.

The old SwiftUI/SwiftData implementation has been replaced. Do not look for `Package.swift`, `Sources/FileSyncMonitor`, SwiftData models, or Swift services in the current codebase.

## STRUCTURE

```text
.
├── package.json                     # npm scripts that delegate to Tauri
├── desktop-portal/
│   ├── index.html                   # Single-page desktop UI
│   ├── main.js                      # Frontend state, Tauri invokes, rendering, i18n hooks
│   ├── styles.css                   # UI theme and layout
│   ├── i18n.js                      # zh/en translation dictionary
│   └── assets/                      # App icon and donation images
├── sync-kernel/
│   ├── Cargo.toml                   # Rust crate and Tauri dependencies
│   ├── tauri.conf.json              # Tauri app config and bundle metadata
│   ├── build.rs
│   ├── capabilities/default.json
│   ├── permissions/default.toml
│   ├── icons/
│   └── src/
│       ├── main.rs                  # Thin executable entry point
│       ├── lib.rs                   # Tauri setup, commands, sync orchestration, login windows
│       ├── db.rs                    # SQLite schema, event/config CRUD
│       ├── monitor.rs               # notify-based recursive file watcher + debounce/coalescing
│       ├── ima_sync.rs              # Tencent IMA API client, upload/download/folders
│       ├── credentials.rs           # IMA credential file storage
│       └── tray.rs                  # Tauri system tray
├── docs/
│   ├── cross_platform_design.md     # Tauri/Rust redesign whitepaper
│   ├── IMA抓包接口总结.md
│   └── screenshot/
└── scripts/build_app.sh
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| App entry point | `sync-kernel/src/main.rs` | Calls `file_sync_monitor_lib::run()` |
| Tauri setup / commands | `sync-kernel/src/lib.rs` | `run()`, `invoke_handler`, app state, login windows, sync orchestration |
| Data schema | `sync-kernel/src/db.rs` | SQLite tables `file_events` and `app_config` |
| File monitor | `sync-kernel/src/monitor.rs` | Rust `notify`, recursive watches, 2s debounce, ignore rules |
| IMA API client | `sync-kernel/src/ima_sync.rs` | Knowledge bases, folders, upload, download, delete, rename, HTTP logs |
| Credentials | `sync-kernel/src/credentials.rs` | Current storage is JSON in Tauri app data dir, with legacy temp migration |
| Tray | `sync-kernel/src/tray.rs` | Tauri tray menu and click behavior |
| Frontend app state | `desktop-portal/main.js` | `state`, boot sequence, rendering, sync actions |
| Main UI markup | `desktop-portal/index.html` | Single-page layout, navigation, panels, dialogs |
| Styling | `desktop-portal/styles.css` | Theme variables and component styles |
| i18n | `desktop-portal/i18n.js` | Translation dictionary for English mode |
| Cross-platform design | `docs/cross_platform_design.md` | Current architecture intent |

## BACKEND MODEL

- `AppState` in `sync-kernel/src/lib.rs` stores:
  - `db_conn: Mutex<rusqlite::Connection>`
  - `monitor: Arc<Mutex<DirectoryMonitor>>`
  - `db_path: PathBuf`
- `FileEvent` in `sync-kernel/src/db.rs` is serialized to the frontend:
  - `id`
  - `path`
  - `old_path`
  - `event_type` (`created`, `modified`, `deleted`, `renamed`)
  - `timestamp`
  - `is_synced`
  - `has_notified`
  - `is_directory`
  - `remote_id`
- App preferences and bindings live in SQLite `app_config`, though the frontend also mirrors some values in `localStorage`.

## FRONTEND MODEL

- `desktop-portal/main.js` is a framework-free SPA.
- It imports `invoke` and `listen` from Tauri globals.
- It keeps a central `state` object with monitored paths, events, credentials, sync status, filters, selected event, knowledge bases, and view modes.
- It renders DOM manually for event trees, pending lists, all records, settings, logs, onboarding, and account status.
- Language is controlled by `localStorage.appLanguage`; translations are looked up from `desktop-portal/i18n.js`.
- Appearance is controlled by `localStorage.appearance` plus the Tauri `set_window_theme` command.

## EVENT FLOW

1. Frontend reads `monitoredDirectories` from SQLite config.
2. Frontend calls `start_file_monitor`.
3. Rust `DirectoryMonitor` starts recursive `notify` watchers.
4. Raw events are filtered by ignore rules and coalesced for 2 seconds.
5. Resolved events are inserted into SQLite.
6. Rust emits `file-change-events`.
7. Frontend calls `get_file_events` and rerenders stats, trees, tables, and details.
8. If auto sync is enabled, frontend calls `sync_all_directories`.

## SYNC FLOW

- `sync_all_directories` performs optional pull and push phases depending on `direction`.
- The file watcher is stopped during sync to avoid feedback loops, then resumed from saved config.
- Pull phase:
  - Fetches all IMA knowledge items.
  - Maps remote folders to local paths.
  - Downloads missing/changed remote files.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LancCJ/file-sync-monitor](https://github.com/LancCJ/file-sync-monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
