---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Emu is a lazygit-inspired TUI for managing Android emulators and iOS simulators, built with Rust. Async-first architecture with trait-based platform abstraction (`DeviceManager`), centralized state (`AppState` with `Arc<Mutex<>>`), and ratatui-based UI.

See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for full architecture details.

## Development Commands

```bash
cargo build                                              # Build
cargo run                                                # Run (default: emu binary)
cargo run -- --check                                     # Verify local environment without opening TUI
cargo run -- --debug                                     # Run with debug logging
cargo check                                              # Type check (fast)
cargo clippy --all-targets --all-features -- -D warnings # Lint (CI-level)
cargo fmt                                                # Format
cargo test --bins --tests                                # Run tests (recommended)
cargo test --features test-utils                         # Run all tests including integration
```

## Key Files

| File                            | Role                                                                                     |
| ------------------------------- | ---------------------------------------------------------------------------------------- |
| `src/app/mod.rs`                | Main event loop shell and app coordination                                               |
| `src/app/state/mod.rs`          | `AppState`, `ApiLevelManagementState::is_busy()`                                         |
| `src/managers/android/mod.rs`   | Android AVD facade and runtime orchestration                                             |
| `src/managers/android/*.rs`     | Android helper modules (`create/details/discovery/install/lifecycle/parser/sdk/version`) |
| `src/managers/ios/mod.rs`       | iOS simulator facade and orchestration (macOS only)                                      |
| `src/managers/ios/*.rs`         | iOS helper modules (`details/discovery/lifecycle/tests`)                                 |
| `src/managers/common.rs`        | `DeviceManager` trait                                                                    |
| `src/app/api_levels.rs`         | API level management mode handling and install/uninstall orchestration                   |
| `src/app/tests.rs`              | `App` orchestration characterization tests and startup fixtures                          |
| `src/app/state/ui.rs`           | UI-facing state enums and confirmation dialog structs                                    |
| `src/app/state/logs.rs`         | Log entry state and log-related `AppState` methods                                       |
| `src/managers/android/tests.rs` | `AndroidManager` facade and helper module regression tests                               |
| `src/managers/ios/tests.rs`     | `IosManager` facade and helper module regression tests                                   |
| `src/ui/render.rs`              | Three-panel layout rendering and panel composition                                       |
| `src/ui/dialogs/mod.rs`         | Modal dialog entrypoint and dialog composition                                           |
| `src/ui/dialogs/*.rs`           | Dialog-specific renderers (`create/confirm/api-level/notifications`)                     |
| `src/ui/panels/mod.rs`          | Panel rendering entrypoint and panel composition helpers                                 |
| `src/ui/panels/*.rs`            | Panel-specific renderers (`device_lists/details/logs/commands`)                          |
| `src/models/device_info/mod.rs` | Dynamic device info entrypoint, cache access, and public surface                         |
| `src/models/device_info/*.rs`   | Device info helpers (`priority/parsing/tests`)                                           |
| `src/constants/`                | All constants (NO hardcoded values in source)                                            |
| `src/models/`                   | Core data structures                                                                     |

## Code Conventions

### String Formatting — CRITICAL

**Always** use inline variable syntax in `format!` and all logging macros:

```rust
// ✅ Correct
format!("Device {name} created")
log::info!("Starting {identifier}")

// ❌ Wrong — clippy::uninlined_format_args error in CI
format!("Device {} created", name)
log::info!("Starting {}", identifier)
```

This applies to `format!`, `println!`, `eprintln!`, `bail!`, `anyhow!`, `log::*`, and test assertions.

### Constants

All hardcoded values must be defined in `src/constants/`. Never use magic numbers or strings in source.

### Error Handling

- `anyhow` for propagation, `thiserror` for custom types
- Never `.unwrap()` or `.expect()` in user-facing code

### Async

- Use `impl Future + Send` for trait methods
- `Arc<Mutex<>>` for shared state; prefer `Arc<AtomicBool>` for simple flags
- Never use `std::sync::Mutex` in async contexts — use `tokio::sync::Mutex`

## Testing


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tomevault-io/codex-plugins](https://github.com/tomevault-io/codex-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
