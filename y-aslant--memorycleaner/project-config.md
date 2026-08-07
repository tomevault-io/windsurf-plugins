---
trigger: always_on
description: Memory Cleaner is a **Windows-only** GUI memory-optimization tool written in Rust with the **GPUI** framework (from the Zed editor). It frees physical and virtual memory by calling Windows NT memory-management APIs (`NtSetSystemInformation`, `SetSystemFileCacheSize`, etc.), runs as a system-tray resident app, and requires administrator privileges for most operations. Licensed MIT.
---

# Repository Guidelines

## Project Overview

Memory Cleaner is a **Windows-only** GUI memory-optimization tool written in Rust with the **GPUI** framework (from the Zed editor). It frees physical and virtual memory by calling Windows NT memory-management APIs (`NtSetSystemInformation`, `SetSystemFileCacheSize`, etc.), runs as a system-tray resident app, and requires administrator privileges for most operations. Licensed MIT.

## Architecture & Data Flow

```
main.rs → ensure_elevated() → single-instance check → notification::init
       → tray install + hotkey::sync → GPUI app launch
 │
 ├─ app.rs (core state, memory refresh, optimization, window hide/restore)
 ├─ log.rs (optional App.log file output, timestamp-based retention)
 ├─ locale.rs (rust-i18n locale apply, list separator, lang-id mapping)
 ├─ memory.rs (GlobalMemoryStatusEx → MemoryStatus)
 ├─ optimize.rs (MemoryAreas bitflags → NT cache-purge steps)
 ├─ settings.rs (TOML persistence at %APPDATA%\MemoryCleaner\settings.toml)
 ├─ privileges.rs (SeProfileSingleProcessPrivilege, SeIncreaseQuotaPrivilege)
 ├─ tray.rs (tray-icon crate, App.png embedded via include_bytes!)
 ├─ icon_cache.rs (Explorer icon cache purge)
 ├─ version.rs (version constant)
 ├─ ui/ (GPUI components: layout, memory_card, settings_page, theme, title_bar)
 └─ win32/ (hotkey, notification, nt, os, process, single_instance, startup, volume, window)
```

- **Entry flow:** `main.rs` → elevation → single-instance mutex → `locale::apply` → `notification::init` → install tray + bind hotkey sender → `hotkey::sync` → GPUI app with `QuitMode::Explicit` → `open_main_window`.
- **i18n:** `rust-i18n` with `locales/zh-CN.yml` (single file, `_version: 2`, zh-CN + en). `rust_i18n::i18n!` is invoked once in `lib.rs`. `settings.language` is `auto` | `zh-CN` | `en`; `auto` uses `GetUserDefaultUILanguage` via `win32::os::system_ui_locale()`. Language changes call `MemoryCleanerApp::apply_locale()` to refresh memory labels and tray menu text immediately.
- **Async runtime:** `smol` for async task execution (optimization progress updates, memory polling, toast display).
- **UI stack:** GPUI + `gpui-component` (Button, Checkbox, Switch, GroupBox, ProgressCircle, Kbd).
- **Native layer:** `src/win32/` wraps low-level Windows APIs; `src/optimize.rs` orchestrates the cleanup steps.
- **Console suppression:** `main.rs` uses `#![windows_subsystem = "windows"]`; diagnostics go to `OutputDebugStringA` (viewable via DebugView). Optional file logging via `src/log.rs` when `debug_logging` is enabled.
- **Tray command channel:** A single `mpsc` channel carries `TrayCommand` from tray events, global hotkeys, and (future) background tasks into `app.rs` via blocking `recv()` — no idle polling loop.
- **Window lifecycle:** Closing with `close_to_notification_area` hides the GPUI window to tray and may destroy the window handle; `activate_window` reopens it. Memory polling pauses while hidden.

## Key Directories

| Path | Purpose |
|---|---|
| `src/` | Application source (binary crate, main.rs entry point) |
| `src/ui/` | GPUI UI components (layout, memory_card, settings_page, theme, title_bar) |
| `locales/` | rust-i18n translation YAML (`zh-CN.yml`, zh-CN + en strings) |
| `docs/` | Project docs (`CHANGELOG.md`, technical comparisons) |
| `src/win32/` | Win32/NT API bindings (hotkey, notification, nt, os, process, single_instance, startup, volume, window) |
| `vendor/proc-macro-error2/` | Vendored patch for Rust 1.97+ compatibility (see below) |
| `.codegraph/` | Codegraph index (gitignored) |

## Development Commands

```bash
# Format
make fmt # cargo fmt

# Lint (clippy with -D warnings — warnings are errors)
make check # cargo clippy -- -D warnings

# Test
make test # cargo test

# Build (release, runs clippy first)
make build # cargo build --release

# Run (debug)
cargo run

# Run (release behavior — console suppressed)
cargo run --release

# Clean
make clean # cargo clean
```

**Tests:** `make test` / `cargo test` — 52 unit tests in `src/` plus 2 integration tests in `tests/settings_persistence.rs`.

## Code Conventions & Common Patterns

- **Language:** Rust, Edition 2024 (requires Rust 1.96+).
- **Platform:** Windows-only. All modules assume `target_os = "windows"`.
- **Error handling:** Functions return `Result<T, E>` or use `Option` for fallible lookups. `anyhow` is used in optimize/icon_cache paths; settings and most UI code use concrete errors.
- **Unsafe / FFI:** `unsafe` is concentrated in `src/win32/` (NT API calls, privilege token manipulation, hotkey message loop) and `src/optimize.rs` (NtSetSystemInformation). Each unsafe block is narrowly scoped.
- **Naming:** Standard Rust conventions — `snake_case` functions/variables, `PascalCase` types, `SCREAMING_SNAKE_CASE` constants. Win32 wrappers match the original API names.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Y-ASLant/MemoryCleaner](https://github.com/Y-ASLant/MemoryCleaner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
