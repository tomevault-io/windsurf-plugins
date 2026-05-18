---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Feedr is a terminal-based RSS/Atom feed reader built with Rust, using ratatui/crossterm for the TUI. It supports feed management, categorization, filtering, dual themes, OPML import, auto-refresh with per-domain rate limiting, feed auto-discovery from HTML pages, configurable keybindings, mouse support, a help overlay, newsboat-style external-command hooks (macros and `exec_on_new`), and Mozilla-Readability full-text article extraction.

## Build & Development Commands

```bash
cargo build --release          # Build optimized binary (LTO enabled)
cargo run --release             # Run the app
cargo test --verbose            # Run all tests
cargo test --all-features --verbose  # Run tests with all features
cargo test <test_name>          # Run a single test
cargo clippy --all-targets --all-features -- -D warnings  # Lint (CI-strict)
cargo fmt --all                 # Format code
cargo fmt --all -- --check      # Check formatting without changing files
```

MSRV: 1.75.0. CI runs tests on stable, beta, and 1.75.0.

## Architecture

**Single-threaded synchronous TUI** — no async runtime. The main loop in `tui.rs` polls for keyboard events, mutates state, then renders.

### Core modules

- **`app.rs`** — `App` struct holding all application state. All state mutations (feed ops, filtering, categorization, persistence) happen through its methods. This is the largest and most central file.
- **`tui.rs`** — Terminal setup/teardown, main event loop (`run_app`), feed refresh logic, and external-command runners (`suspend_for_command`, `spawn_detached`, `drain_macro_steps`, `collect_exec_on_new`/`flush_exec_on_new`). `TerminalRestoreGuard` (RAII) re-enters alt-screen + raw mode + mouse capture on drop so a panic in a child invocation can't leave the terminal broken.
- **`events.rs`** — All keyboard and mouse event handling (`handle_events`). Input dispatches based on `View` × `InputMode` enums. Hosts the macro engine (`dispatch_action`, `run_macro`, `build_pipe_invocation`, `build_exec_invocation`). Separated from `tui.rs` for maintainability.
- **`keybindings.rs`** — `KeyAction` enum, default keybinding map, key string parsing, config-driven keybinding overrides via `[keybindings]` TOML section, and newsboat-style macro parsing (`MacroStep`, `MacroBinding`, `MacroOptions`, `parse_macro_string`).
- **`feed.rs`** — Data models (`Feed`, `FeedItem`, `FeedCategory`), RSS/Atom parsing via `feed-rs`, and HTML feed auto-discovery via `scraper`.
- **`config.rs`** — XDG-compliant config loading/saving (`~/.config/feedr/config.toml`). Includes `keybindings: HashMap<String, toml::Value>` for custom key overrides, `[hooks]` (`exec_on_new`), `[macros]`, and `[macro_options]`. Auto-generates defaults on first run.
- **`config_cli.rs`** — CLI subcommand handler for `feedr config list/get/set`.
- **`config_tui.rs`** — Interactive TUI config editor (`feedr config --tui`).
- **`config_ui.rs`** — Rendering for the TUI config editor.
- **`main.rs`** — CLI arg parsing (clap) and OPML import entry point.

### UI modules (`src/ui/`)

- **`mod.rs`** — Rendering dispatch, `ColorScheme` with two themes (dark cyberpunk, light zen), and shared layout helpers.
- **`dashboard.rs`** — Dashboard view with filters, search, and preview pane.
- **`feed_list.rs`** — Feed list and hierarchical tree view rendering.
- **`feed_items.rs`** — Feed items list rendering.
- **`detail.rs`** — Article detail view with scrolling and link extraction.
- **`starred.rs`** — Starred articles view.
- **`categories.rs`** — Category management UI.
- **`summary.rs`** — Session summary ("What's New") screen.
- **`modals.rs`** — Error, input, filter, link overlay, and help overlay modals.
- **`utils.rs`** — Shared rendering utilities.

### Key patterns

- **View + InputMode dispatch**: Event handling in `events.rs` matches on `(app.view, key.code)` nested inside `app.input_mode`. When adding new keybindings, place them in the correct View/InputMode branch.
- **Configurable keybindings**: All remappable actions are defined as `KeyAction` variants in `keybindings.rs`. The `KeyBindingMap` is built from defaults merged with user overrides from `config.keybindings`. Event handlers use `app.keybindings` to check matches instead of hardcoding key codes. Some structural keys (Tab/Shift+Tab, number keys, text input, category/filter mode keys) are intentionally hardcoded.
- **`q` key goes back, not quit**: `q` navigates back one view (e.g., FeedItems → FeedList → Dashboard). Only quits from Dashboard. `Ctrl+Q` is the universal quit from any view. The `Ctrl+Q` check is a guard at the top of `handle_events`, before the `match app.input_mode` block.
- **Feed auto-discovery**: When a user adds a non-RSS URL, `feed.rs` fetches the HTML and uses `scraper` to find `<link>` tags with RSS/Atom types. If feeds are found, a confirmation dialog lets the user pick which to subscribe to.
- **Mouse support**: `events.rs` handles `MouseEventKind::Down` (left click to select) and `MouseEventKind::ScrollDown`/`ScrollUp` for navigation.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bahdotsh/feedr](https://github.com/bahdotsh/feedr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
