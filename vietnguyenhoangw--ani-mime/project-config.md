---
trigger: always_on
description: A floating desktop mascot (pixel dog) that reacts to terminal and Claude Code activity in real-time. Ships on macOS; runs on Linux and WSL2 via a cross-platform backend facade. Built with Tauri 2 + React 19.
---

# Ani-Mime

A floating desktop mascot (pixel dog) that reacts to terminal and Claude Code activity in real-time. Ships on macOS; runs on Linux and WSL2 via a cross-platform backend facade. Built with Tauri 2 + React 19.

<!-- c3-generated: c3-0 -->
## Architecture (C3)

This project uses C3 architecture docs in `.c3/`. For architecture questions, changes, audits, or file ownership lookup, invoke `/c3-skill:c3` with one of: query, audit, change, ref, rule, sweep.

- File → component lookup: `c3x lookup <file-or-glob>` (use the c3-skill bin)
- Topology: `c3x list` (containers, components, refs, rules)
- Validate: `c3x check` (must be zero issues before merging architecture changes)
- HARD RULE: never read or edit files under `.c3/` directly — always go through `c3x` CLI

Containers: `c3-1` Rust Backend, `c3-2` React Frontend, `c3-3` Shell Integration. See `.c3/README.md` (via `c3x read c3-0`) for the system goal and the rules cited from each component.
<!-- /c3-generated -->

## Quick Reference

- **Dev**: `bun run tauri dev`
- **Build (macOS)**: `bun run tauri build && bash src-tauri/script/post-build-sign.sh`
- **Build (Linux/WSL2)**: `bun run tauri build` — produces AppImage + .deb in `src-tauri/target/release/bundle/`
- **Type check frontend**: `npx tsc --noEmit`
- **Type check backend**: `cd src-tauri && cargo check`
- **Package manager**: Bun (not npm/yarn)
- **Entitlements (macOS)**: `src-tauri/Entitlements.plist` (network + Hardened Runtime); post-build re-sign is required for ad-hoc builds
- **Linux prerequisites**: `sudo apt install libwebkit2gtk-4.1-dev libgtk-3-dev libglib2.0-dev libayatana-appindicator3-dev librsvg2-dev zenity` — required for `cargo check` and runtime dialogs

## Architecture

See `docs/ARCHITECTURE.md` for full details. Key data flow:

```
Shell hooks (curl) → HTTP :1234 → Rust state → Tauri event → React UI
Claude Code ←stdio→ MCP server (Node.js) ←HTTP→ :1234 → Tauri event → React UI
Peer announces ←UDP :1235 (mDNS + multicast + unicast scan)→ AppState.peers → peers-changed event → UI
```

### Backend (`src-tauri/src/`)

| Module | Responsibility |
|--------|---------------|
| `lib.rs` | Tauri setup, plugin registration, tray icon, composition root, all `#[tauri::command]` functions |
| `state.rs` | `AppState`, `Session`, `SessionInfo`, `resolve_ui_state()`, `emit_if_changed()` |
| `server.rs` | HTTP server on `127.0.0.1:1234` (tiny_http), incl. MCP endpoints. Rejects status/heartbeat with 410 when the pid isn't alive |
| `watchdog.rs` | Background thread: service→idle transition, stale session cleanup |
| `proc_scan.rs` | Background thread (2s): libproc-based OS scan — auto-discovers shells, fills `pwd`/`tty`/`fg_cmd`, detects `claude` via `KERN_PROCARGS2` argv, drops zombie sessions |
| `focus.rs` | `focus_terminal_for_pid()` — walks parent chain to find owning terminal app (iTerm/Terminal/VS Code/Cursor/tmux/etc.), activates via `open -a`, optionally targets tab via AppleScript |
| `discovery.rs` | mDNS peer discovery — registers `_ani-mime._tcp.local.`, browses peers, emits `peers-changed` |
| `broadcast.rs` | UDP peer discovery on `:1235` — multicast announce (224.0.0.200 every 5s), unicast `/24` scan (every 30s), receive loop, expiry loop. Writes into the same `AppState.peers` as `discovery.rs` keyed by `instance_name` |
| `helpers.rs` | `now_secs()`, `get_query_param()` |
| `setup/mod.rs` | First-launch auto-setup orchestrator |
| `setup/shell.rs` | Shell detection, RC file injection, shell-selection prompt (via `platform::show_dialog`) |
| `setup/claude.rs` | Claude Code hooks configuration |
| `setup/mcp.rs` | MCP server installation + Claude Code MCP registration |
| `logger.rs` | Log file tail-reader, `app_log!`/`app_warn!`/`app_error!` macros |
| `platform/mod.rs` | Cross-platform facade: `setup_main_window`, `set_dock_visibility`, `open_path`, `open_url`, `show_dialog`, `show_choose_list`, `open_local_network_settings`, `run_update_command` |
| `platform/macos.rs` | macOS impl: Cocoa/objc window tweaks, `osascript` dialogs, `open` / `brew --cask` update flow |
| `platform/linux.rs` | Linux impl: Tauri-native transparency, `zenity` dialogs, `xdg-open`, manual update via release URL |
| `plugin/runtime.rs` | Spawns a per-plugin WebView (`plugin://<id>/<entry>`), injects the `window.ani` SDK, maps plugin id ↔ `plugin-<id>` window label |
| `plugin/gateway.rs` | `plugin_call` command — the single gated entry point for `window.ani`; checks declared capabilities, dispatches to storage / window ops |
| `plugin/storage.rs` | Per-plugin key/value store at `~/.ani-mime/plugins/<id>/data/store.json` |
| `plugin/clipboard.rs` | Background OS-clipboard monitor (arboard) + history (deduped, capped 20, persisted to `~/.ani-mime/clipboard-history.json`); runs only while a `clipboard`-capable plugin is enabled; exposed via the `clipboard` capability |

### MCP Server (`src-tauri/mcp-server/`)

| File | Responsibility |
|------|---------------|
| `server.mjs` | Zero-dependency Node.js MCP server (JSON-RPC 2.0 over stdio) |

### Frontend (`src/`)

| Module | Responsibility |
|--------|---------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vietnguyenhoangw/ani-mime](https://github.com/vietnguyenhoangw/ani-mime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
