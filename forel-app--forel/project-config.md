---
trigger: always_on
description: > **Source of truth for all AI coding agents** (Claude Code, Cursor, Copilot, etc.).
---

# Forel — AI & Agent Guidelines

> **Source of truth for all AI coding agents** (Claude Code, Cursor, Copilot, etc.).
> `CLAUDE.md` is a symlink to this file — edit only `AGENTS.md`.

---

## What Forel is

Forel is an **open-source macOS file-automation app** (think Hazel). It watches folders and runs user-defined rules (conditions → actions) on new/changed files. It lives in the system tray and applies rules silently in the background.

**Status: alpha.** Core plumbing works; many planned features are not yet implemented.

---

## Stack

| Layer | Technology |
|---|---|
| App shell | Tauri 2 |
| Backend | Rust (stable) |
| Frontend | React 19 + TypeScript |
| State (UI) | Zustand 5 |
| Persistence | SQLite via `rusqlite` (bundled) |
| File watching | `notify` 6 (FSEvents on macOS) |
| macOS tags | `xattr` + `plist` crates |
| Icons | `lucide-react` |
| Build | Vite 7 + `pnpm` |

---

## The IPC boundary — the most important constraint

Tauri enforces a **hard process boundary** between the React frontend (WebView) and the Rust backend. They communicate exclusively through **typed IPC commands**.

```
React (WebView)                   Rust (native process)
──────────────────                ────────────────────────────────
invoke("command_name", args)  →   #[tauri::command] fn command_name(...)
        ↑                                   |
        └──────────── JSON response ────────┘
```

**Every frontend feature that reads or mutates app state requires a matching Rust command.**
There is no shared memory, no filesystem shortcut, no hidden channel.

When adding a feature, ask yourself: _"Does the frontend need data from the OS/DB, or does it need to trigger a side effect?"_ If yes → you need a Rust command.

### Adding a command — the full checklist

1. **`src-tauri/src/commands.rs`** — write the `#[tauri::command]` function.
   - Accept `state: State<AppState>` for DB/watcher access.
   - Accept `app: AppHandle` if you need to rebuild the tray afterward.
   - Return `Result<T, String>` — errors become JS `Promise` rejections.

2. **`src-tauri/src/lib.rs`** — register it in `invoke_handler!`:
   ```rust
   commands::your_new_command,
   ```
   A command not listed here is invisible to the frontend — no build error, it just silently fails.

3. **`src/store/index.ts`** — add a method to `useForelStore` that calls `invoke<ReturnType>("your_new_command", { arg })`.

4. **`src/types/index.ts`** — add or extend types if the command returns new shapes.

5. **`src-tauri/capabilities/default.json`** — if your command touches a Tauri plugin (fs, dialog, etc.), add the required permission here.

---

## Repository layout

```
forel/
├── src/                          React frontend
│   ├── App.tsx                   Root: layout, sidebar, rule list
│   ├── App.css                   All styles (single file, no CSS modules)
│   ├── main.tsx                  React entry point
│   ├── components/
│   │   ├── RuleEditor.tsx        Modal: edit conditions + actions for a rule
│   │   ├── RuleList.tsx          Right panel: list rules for selected folder
│   │   └── Sidebar.tsx           Left panel: watched folders
│   ├── store/
│   │   └── index.ts              Zustand store — all invoke() calls live here
│   └── types/
│       └── index.ts              Shared TS types + UI label maps
│
└── src-tauri/                    Rust backend (Tauri 2)
    ├── Cargo.toml
    ├── tauri.conf.json           App metadata, window config, bundle ID
    ├── capabilities/
    │   └── default.json          Tauri permission grants
    └── src/
        ├── lib.rs                App setup: DB init, watcher start, tray, IPC registration
        ├── main.rs               Binary entry (calls lib::run)
        ├── state.rs              AppState struct (db, watcher, paused flag)
        ├── commands.rs           All #[tauri::command] functions
        ├── db.rs                 SQLite schema + all query helpers
        ├── tray.rs               System tray icon, menu, event handler
        ├── watcher.rs            notify-based FSEvents loop
        └── rules/
            ├── mod.rs            Re-exports
            ├── model.rs          Rule/Condition/Action types (serde ↔ DB)
            ├── condition.rs      Condition evaluation logic
            ├── action.rs         Action execution logic (move, tag, script…)
            └── engine.rs         Applies rules to a file path
```

---

## Dev commands

```bash
# Run app in dev mode (hot-reload frontend, Rust recompiles on change)
pnpm tauri dev

# Type-check frontend only
pnpm build          # tsc + vite build

# Check Rust without linking (fast)
cargo check         # run from src-tauri/

# Full Rust build (slow, needed before tauri dev first run)
cargo build         # run from src-tauri/

# Regenerate all icon sizes from a square PNG source
pnpm tauri icon assets/forel-icon.png

# Package the app (.dmg / .app)
pnpm tauri build
```

> `pnpm` is required (`npm` and `yarn` are not used). Run all JS commands from the repo root.
> Run `cargo` commands from `src-tauri/`.

---

## Compilation warnings policy

**Every `cargo check` and `cargo build` run must finish with zero warnings introduced by your change.**

Run this before considering any Rust work done:

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [forel-app/forel](https://github.com/forel-app/forel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
