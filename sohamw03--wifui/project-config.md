---
trigger: always_on
description: This document is the quick orientation guide for contributors and coding agents working in WifUI. The user-facing overview and installation instructions remain in [README.md](README.md).
---

# WifUI Codebase Index and Support Guide

This document is the quick orientation guide for contributors and coding agents working in WifUI. The user-facing overview and installation instructions remain in [README.md](README.md).

## Project status

WifUI is a Rust terminal UI for Wi-Fi management.

- Windows uses the native Windows WLAN API.
- Linux builds and launches the TUI with a runtime-selected NetworkManager or iwd D-Bus backend.
- Linux initially targets the first usable Wi-Fi interface. NetworkManager is preferred in `auto` mode.
- Linux supports saved-profile auto-connect toggling and profile deletion through the selected daemon.
- Other non-Windows targets compile against an unsupported placeholder backend.
- The frontend talks only to the platform facade in `src/wifi/mod.rs`; UI code should not import Windows APIs.

## Quick commands

Run these from the repository root:

```sh
cargo fmt -- --check
cargo check --all-targets
cargo test
cargo run -- --ascii
```

For a normal Linux development environment, the same checks can be run offline when dependencies are already cached:

```sh
cargo check --offline --all-targets
cargo test --offline
```

`--ascii` avoids requiring a Nerd Font while testing the TUI. The application expects an interactive terminal; use a PTY when testing launch, key handling, or rendering.

## Architecture at a glance

```text
src/main.rs
  ├─ creates AppState
  ├─ initializes the selected Linux backend and starts the initial refresh when available
  └─ initializes the terminal and enters event::run

src/event/mod.rs
  ├─ draws the UI
  ├─ initializes the Windows listener when applicable
  ├─ coordinates background scans, refreshes, and connection results
  └─ dispatches key events to src/event/handlers.rs

src/ui.rs
  └─ renders the shared TUI from AppState

src/wifi/mod.rs
  └─ selects the platform backend at compile time and re-exports the stable API
       └─ Linux dispatches to NetworkManager or iwd over system D-Bus
```

The normal runtime flow is:

1. `main` creates an empty `AppState`.
2. Windows starts an initial scan and connected-SSID query. Unsupported platforms skip this work and clear the startup loading state immediately.
3. `event::run` renders frames, polls input, handles background results, and performs Windows automatic refreshes.
4. `ui::render` displays the network list, details, popups, and the shared error panel.

## Source index

| Path | Responsibility |
| --- | --- |
| `Cargo.toml` | Package metadata, shared dependencies, and target-specific Windows/Linux dependency selection |
| `Cargo.lock` | Resolved dependency versions; retain Windows entries even on Linux |
| `src/main.rs` | CLI arguments, terminal setup/restore, initial backend refresh |
| `src/app.rs` | `AppState` and the network, UI, connection, input, and refresh state models; refresh application with selection preservation |
| `src/config.rs` | UI dimensions, timing constants, refresh burst sizes, and icons |
| `src/error.rs` | `WifiError`, `WifiResult`, and Windows WLAN reason-code formatting |
| `src/event/mod.rs` | Main async event loop, background task result handling, listener setup |
| `src/event/handlers.rs` | Keyboard handlers, connection/profile actions, search, and QR generation |
| `src/input.rs` | Editable input state and cursor/word navigation |
| `src/ui.rs` | Ratatui rendering split into per-panel functions (list, details, popups) with shared input-scrolling and spinner helpers |
| `src/theme.rs` | Shared TUI colors and styles |
| `src/wifi/mod.rs` | Platform facade and compile-time backend selection |
| `src/wifi/types.rs` | Shared `WifiInfo` and `ConnectionEvent` data types, plus network-list merge/sort helpers shared by all backends |
| `src/wifi/connection.rs` | Windows connect, disconnect, connected-SSID, and network-list operations |
| `src/wifi/profile.rs` | Windows profile XML, saved profiles, passwords, auto-connect, and forget operations |
| `src/wifi/scanning.rs` | Windows scan trigger |
| `src/wifi/listener.rs` | Windows WLAN notification listener |
| `src/wifi/handle.rs` | Safe Windows WLAN handle wrapper |
| `src/wifi/linux.rs` | Linux runtime registry, backend choice, dispatcher, and stable API |
| `src/wifi/linux_network_manager.rs` | NetworkManager system-D-Bus adapter and conversion helpers |
| `src/wifi/linux_iwd.rs` | iwd system-D-Bus adapter, conversion helpers, and temporary credential agent |
| `src/wifi/linux_listener.rs` | Long-lived Linux D-Bus signal worker and shutdown guard |
| `src/wifi/unsupported.rs` | Same placeholder contract for other unsupported targets |
| `dist-workspace.toml` | cargo-dist release targets and installer configuration |
| `wix/main.wxs` | WiX installer template for Windows |

## Wi-Fi backend boundary

`src/wifi/mod.rs` is the only backend boundary. It always compiles the shared types, then selects modules with target configuration:

| Target | Backend | `is_backend_available()` |
| --- | --- | --- |
| Windows | `connection`, `handle`, `listener`, `profile`, `scanning` | `true` |
| Linux | runtime NetworkManager or iwd adapter | `true` after successful initialization |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sohamw03/wifui](https://github.com/sohamw03/wifui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
