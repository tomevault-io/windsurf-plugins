---
trigger: always_on
description: Notes for Claude (and humans) working on this repo.
---

# CLAUDE.md

Notes for Claude (and humans) working on this repo.

## What this is

Cross-platform (macOS, Windows, Linux) desktop client for the public **JLab static
JAR scanner** at `https://jlab.threat.rip/api/public/static-scan`. User drops
a `.jar` (or a `.zip` / `.mcpack` / `.mrpack` containing one), the app
uploads it as `multipart/form-data`, and renders the matched signatures
grouped by severity. For container drops, the Rust side opens the archive,
picks the largest inner `.jar`, and uploads only that file.

Open-source, dual-licensed **MIT OR Apache-2.0**. Repo:
`https://github.com/NeikiDev/jlab-desktop`.

## Stack

- **Shell:** Tauri 2 (Rust)
- **Frontend:** React 19 + TypeScript + Vite, styled with Tailwind v4 (tokens defined in `@theme` inside `src/index.css`, wired up via `@tailwindcss/vite`, no `tailwind.config.js`, no PostCSS config)
- **HTTP:** `reqwest` 0.12 with `multipart` + `rustls-tls` (no OpenSSL)
- **Async runtime:** `tokio`
- **Errors:** `thiserror` + `serde(tag = "kind")` so the frontend can discriminate

The HTTP upload runs in **Rust**, never in the webview. File bytes don't
cross the IPC boundary as JSON. The frontend only passes the file path.

## Layout

```
src/                    React frontend
  main.tsx              createRoot mount, imports ./index.css
  App.tsx               State machine (idle | scanning | result | error)
                        plus showingHistory / showingWatcher routing
  index.css             Tailwind v4 entry + @theme tokens (dark only),
                        keyframes, prefers-reduced-motion override
  lib/
    api.ts              invoke() wrappers + AppError type guards
    types.ts            API JSON schema + watcher types
    cn.ts               Six-line className concat helper
    components/         DropZone, ScanProgress, SignatureList,
                        SignatureCard, SeverityBadge, ErrorBanner,
                        RemoteStatus, FamilyAlert, IdleDashboard,
                        WatcherPanel, WatcherStatusCard,
                        WatcherFoldersList, WatcherSettingsList,
                        WatcherFirstEnableModal, WatcherIcons
src-tauri/              Rust shell
  src/
    main.rs             Thin entry; calls lib::run()
    lib.rs              tauri::Builder, plugin + command registration,
                        watcher store + tray + close-requested hook +
                        autostart reconcile
    api.rs              scan_jar (manual command), run_scan (shared
                        helper for manual + watcher), check_status,
                        open_url, history commands
    error.rs            AppError enum (tagged, serializable)
    history.rs          history.json store (atomic writes, schema v1)
    paths.rs            Friendly per-OS data + log dir resolver
    watcher/            Folder watcher subsystem (see below)
  capabilities/         Tauri 2 permission scopes
  tauri.conf.json       Bundle config (DMG/APP/MSI/NSIS), CSP, window
TODO/                   Tracked via /todo skill
```

### Folder watcher (`src-tauri/src/watcher/`)

Opt-in. Off by default. Reuses the manual `scan_jar` pipeline via
`api::run_scan(.., source: ScanSource)`.

```
watcher/
  mod.rs        // declares submodules + re-exports WatcherStore
  settings.rs   // WatcherSettings, atomic JSON store, validate_watch_path
  core.rs       // WatcherStore, debouncer, mpsc queue, 12 req/min
                // token bucket, process_one, event emitter
  hold.rs       // rename foo.jar <-> foo.jar.jlab-pending
                // recover_stragglers() on startup
  trash.rs      // OS trash via the `trash` crate
  quarantine.rs // move to <data_dir>/quarantine/<ts>-<name>.quarantined
  rescan.rs     // 6 h scheduler, re-queues stale jars (7 / 14 / 30 d)
  notify.rs     // coalesced native notifications (4 s window)
  tray.rs       // TrayIconBuilder + menu (open / pause / quit)
  commands.rs   // #[tauri::command] entry points
```

Settings file: `watcher-settings.json` next to `history.json` in
`paths::friendly_data_dir()`. Schema `version: 1` with `#[serde(default)]`
on additive fields so older files migrate cleanly. The `auto_action`
field also has `#[serde(alias = "autoDelete")]` so settings written
under the previous name still load.

Quarantine: `<data_dir>/quarantine/` (lazy-created, `0o700` on Unix).
Files land as `<unix-timestamp>-<original-name>.quarantined`. The suffix
prevents Java launchers from loading them and stops external AV from
auto-classifying them as live `.jar` payloads. Rename first, copy +
delete fallback for cross-volume cases.

Auto-action is two fields:
- `auto_action: ActionThreshold` — `off` | `multiple_criticals` |
  `confirmed_families_only`. Default `multiple_criticals`.
- `auto_action_mode: ActionMode` — `quarantine` | `trash`. Default
  `quarantine` (per-user request: prefer recoverable quarantine over
  trash).

`WatcherEvent::ScanCompleted.action: Option<String>` carries
`"quarantined"` / `"trashed"` / `null` so the frontend renders the
correct chip on the recent-hits row.

Hold-until-scanned: while a scan runs, the file is renamed in place to
add `.jlab-pending`. After the scan the suffix is removed (or
auto-action runs). On app start `hold::recover_stragglers` re-queues any

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NeikiDev/jlab-desktop](https://github.com/NeikiDev/jlab-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
