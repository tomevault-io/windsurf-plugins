---
trigger: always_on
description: **Windows-only** clipboard history replacement currently built with Rust + Tauri 2.x + React + TypeScript. Tauri remains under evaluation against WinUI 3. Do not add macOS/Linux product work.
---

# Cubby Development Guide

**Windows-only** clipboard history replacement currently built with Rust + Tauri 2.x + React + TypeScript. Tauri remains under evaluation against WinUI 3. Do not add macOS/Linux product work.

## Project Structure

```
Cubby/
├── src-tauri/src/
│   ├── lib.rs               # App bootstrap, cursor positioning, tray, hotkey, blur handler
│   ├── commands.rs          # Tauri IPC commands (clips, paste, folders, shortcuts)
│   ├── settings_commands.rs # Settings IPC (get_settings, save_settings, ignored apps)
│   ├── clipboard.rs         # Clipboard polling loop, content capture
│   ├── database.rs          # SQLite via sqlx (clips, folders, settings tables)
│   ├── models.rs            # Shared types + global tokio runtime (get_runtime())
│   ├── settings_manager.rs  # In-memory settings cache with DB persistence
│   ├── constants.rs         # Flyout dimensions, cursor offset, and monitor margin
│   └── main.rs              # Entry point
├── frontend/src/
│   ├── App.tsx              # Root component, keyboard shortcuts, IPC calls
│   ├── components/          # FlyoutHeader, ClipList, ClipCard, SettingsPanel, ...
│   ├── hooks/               # useClips, useSearch, useKeyboard, ...
│   ├── types/index.ts       # Shared TS types
│   └── constants.ts         # WINDOW_HEIGHT, LAYOUT constants
└── .github/workflows/release.yml  # CI: builds x64 + arm64 NSIS installers
```

## Architecture & Key Systems

### Window Show/Hide State Machine
All show/hide goes through `lib.rs`. Three global atomics guard it:
- `IS_ANIMATING: AtomicBool` — prevents concurrent animations. Both `animate_window_show` and `animate_window_hide` use `compare_exchange(false, true)` at entry and set back to `false` on exit.
- `LAST_SHOW_TIME: AtomicI64` — timestamp set on show; blur events within 500ms are ignored to prevent immediate re-hide.
- `SHOW_GENERATION: AtomicU64` — invalidates the native outside-click watcher whenever a newer flyout invocation starts.

`position_window_near_cursor()` is the public entry point — it calls `animate_window_show()`.

**Hotkey toggle logic** (both in `setup` and in `register_global_shortcut`):
```rust
if win.is_visible().unwrap_or(false) && win.is_focused().unwrap_or(false) {
    animate_window_hide(&win, None);
} else {
    position_window_near_cursor(&win);
}
```
Both places must have identical toggle logic. Issue #6 was caused by `register_global_shortcut` missing the toggle.

### Cursor-anchored flyout
The main window is a compact flyout. `animate_window_show` reads the physical
cursor position and centers the flyout horizontally around it. It opens at full
height below the cursor when possible, then flips to full height above the cursor
when there is not enough room below. It shortens vertically on the roomier side
only when full height fits on neither side. The cursor stays just outside the
nearest vertical edge, and the window clamps to the monitor work area without
moving the cursor. Do not restore full-monitor shelf sizing.

### Blur → Auto-hide
`on_window_event` → `Focused(false)` → skips if: settings window is open,
`LAST_SHOW_TIME` debounce < 500ms, `IS_ANIMATING` is true, or window is already
hidden. Otherwise it hides immediately. A generation-scoped native mouse watcher
also hides the main flyout when any mouse button is pressed outside its window
rectangle. This is required because Windows can deny the initial focus handoff,
leaving no later blur event. The next invocation repositions near the current
cursor.

### Settings
`SettingsManager` is managed state (`app.manage(Arc::new(settings_manager))`). Access via `window.state::<Arc<SettingsManager>>().get()`. Persisted to DB. Settings changes that affect hotkey require calling `commands::register_global_shortcut` which unregisters the old shortcut and re-registers with the toggle logic.

### IPC (Frontend → Backend)
All commands are registered in `lib.rs` `invoke_handler!`. Frontend calls via `invoke("command_name", args)`. Commands return `Result<T, String>`.

### Feature Flags
- `app-store` feature: disables `tauri-plugin-autostart` and `tauri-plugin-updater` (not applicable to Windows builds, but keep the `#[cfg(not(feature = "app-store"))]` guards).

### Window Effects
`apply_window_effect(window, effect, theme)` wraps `window_vibrancy` crate:
- `"mica"` / `"dark"` → `apply_mica`
- `"mica_alt"` / `"auto"` / default → `apply_tabbed`
- `"clear"` → `clear_mica`
Re-applied on system theme change if user setting is `"system"`.

### CI / Release
- Builds both `x86_64-pc-windows-msvc` and `aarch64-pc-windows-msvc`
- winget installer regex: `.*-setup\.exe$` (NSIS only — WiX MSIs have x64 bootstrap stubs that fool komac's arch detection)
- Triggered by `v*` tags; `workflow_dispatch` creates a draft prerelease

## Build Commands

```bash
# Full dev (hot reload)
pnpm tauri dev

# Production build
pnpm tauri build

# Rust only
cargo check          # fast error check
cargo clippy         # lint
cargo fmt            # format
cargo test           # tests

# Frontend only (in frontend/)
pnpm install && pnpm dev

# Frontend tests (vitest, from the repo root)
pnpm run test
```

## Code Style

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [btsouth/cubby-clipboard](https://github.com/btsouth/cubby-clipboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
