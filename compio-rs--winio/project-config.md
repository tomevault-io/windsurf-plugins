---
trigger: always_on
description: This file provides guidance for AI agents working in this repository.
---

# AGENTS.md

This file provides guidance for AI agents working in this repository.

## Project overview

winio is a single-threaded asynchronous GUI framework for Rust, based on [compio](https://github.com/compio-rs/compio) (an async runtime built on io-uring and IOCP) and an ELM-style architecture. It provides native widgets across desktop and mobile platforms through a common API.

The framework follows the Elm Architecture:

- **Components** (`winio-elm::Component`) hold UI state and react to **messages**; they emit **events** to their parent.
- Messages flow down the component tree; events flow up.
- A component owns its children as `Child<T>` fields.
- `Root<T>` runs a component tree and yields events.

## Repository layout

This is a Cargo workspace with these crates:

| Crate                     | Purpose                                                                                                           |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| `winio`                   | The public API: widgets, UI helpers, backend selection (`sys`)                                                    |
| `winio-elm`               | ELM primitives: `Component`, `Child`, `Root`, `Prop`/`PropSource`, macros (`init!`, `start!`, `update_children!`) |
| `winio-primitive`         | Basic types: `Point`, `Size`, `Rect`, `Color`, `Font`, `HAlign`, `VAlign`, ...                                    |
| `winio-layout`            | Layout containers (`Grid`, `StackPanel`) and the `layout!` macro                                                  |
| `winio-handle`            | Native handle abstractions (`AsWindow`, `AsWidget`, `AsContainer`)                                                |
| `winio-callback`          | Callback helpers                                                                                                  |
| `winio-pollable`          | Runtime-polling helpers                                                                                           |
| `winio-example`           | The example application, exercising all widgets                                                                   |
| `winio-ui-qt`             | Qt backend                                                                                                        |
| `winio-ui-gtk`            | GTK 4 backend                                                                                                     |
| `winio-ui-app-kit`        | macOS (AppKit) backend                                                                                            |
| `winio-ui-ui-kit`         | iOS (UIKit) backend                                                                                               |
| `winio-ui-win32`          | Win32 backend                                                                                                     |
| `winio-ui-winui`          | WinUI 3 backend                                                                                                   |
| `winio-ui-android`        | Android backend                                                                                                   |
| `winio-ui-stub`           | No-op stub backend (fallback)                                                                                     |
| `winio-ui-apple-common`   | Shared code for the Apple backends                                                                                |
| `winio-ui-windows-common` | Shared code for the Windows backends                                                                              |

The active backend is selected in `winio/src/lib.rs` with features and `cfg` attributes (e.g. `win32` feature *on Windows* prefers `winio-ui-win32` over `winio-ui-winui`, and `qt` feature *on Linux* prefers `winio-ui-qt` over `winio-ui-gtk`). `sys` is the platform backend alias used throughout the `winio` crate. On Windows or Linux, if a backend is not selected by the feature, `winio-ui-stub` will be the fallback.

## Common commands

Commands are grouped by host OS.

The code should be formatted after changes.
```sh
cargo fmt
```

WinUI bindings are manually generated and published. If a WinRT API is missing, report it instead of working around it.

### Linux

The reference environment for this checkout is WSL, with Qt 6 dev packages, `cargo xwin` (Windows), `cargo ndk` (`ANDROID_NDK_HOME`, Android) and the Apple targets (through `osxcross`) installed.

```sh
# Default platform (Qt backend on Linux)
cargo clippy --workspace --features all,nightly

# GTK backend
cargo clippy --no-default-features --features all,gtk,nightly

# Stub backend
cargo clippy --no-default-features --workspace --features all,nightly

# Win32 (MSVC target via xwin)
cargo xwin clippy --target x86_64-pc-windows-msvc --workspace --features all,nightly,windows-dark-mode

# Win32 (MSVC target via xwin)
cargo xwin clippy --target x86_64-pc-windows-msvc --no-default-features --workspace --features all,nightly,windows-dark-mode,winui,winui-enable-cbs,winui-webview-system,winui-content-dialog

# macOS
cargo clippy --target x86_64-apple-darwin --workspace --features all,nightly


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [compio-rs/winio](https://github.com/compio-rs/winio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
