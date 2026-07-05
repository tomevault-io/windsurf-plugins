---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build, run, test

```sh
cargo build --release          # release binary lands at target/release/binvim
cargo test                     # ~207 unit tests, all in `mod tests` blocks beside the code they cover
cargo test motion::tests       # run a single module's tests
cargo test motion::tests::word_forward_basic   # single test
cargo run -- path/to/file      # debug-build run; `binvim [path]` once installed
```

The user's `binvim` shell alias points at `target/release/binvim`, so any change you want them to exercise interactively needs a fresh `cargo build --release` — debug-build behaviour will not be picked up by their alias. Mention this in the hand-off if you've only built debug.

CI runs `cargo test`, `cargo clippy` (warnings allowed for now), and `cargo fmt --check`. The fmt gate is configured by `rustfmt.toml` at the repo root: `max_width = 100` + `single_line_let_else_max_width = 100` to preserve compact `let Some(x) = … else { return; };` and single-line method chains. Run `cargo fmt` before pushing — anything that doesn't fit the config will fail CI.

## Architecture

binvim is a single-binary modal TUI editor. `main.rs` does almost nothing — it parses one optional path argument and hands control to `App::run()`. Most modules live flat under `src/`; the three largest — `app`, `lsp`, and `dap` — are sub-module directories. The parent file (`src/app.rs` / `src/lsp.rs` / `src/dap.rs`) declares the children and re-exports the public API, so external callers still address `crate::app::App` / `crate::lsp::LspManager` / `crate::dap::DapManager` directly.

- **`app.rs` + `app/`** is the heart — `App` owns the event loop, the active buffer + per-buffer stashes (`BufferStash`), all transient UI state (completion popup, hover, signature help, which-key, pickers, command line), and the dispatch from parsed `Action`s into mutations. The struct definition + `new`/`run`/`TerminalGuard` live in `app.rs`; everything else is split across `app/<topic>.rs` files (each contains `impl super::App { … }` blocks). The map: **`state`** (UI types, constants, small helpers), **`pair`** (bracket and HTML-tag matching, auto-pair), **`view`** (viewport, scrolling, folds, highlight cache), **`search`** (search, jumps, per-line range queries), **`registers`** (registers, macros, `.` repeat, clipboard mirror), **`buffers`** (open/switch/delete + recents + disk reload), **`save`** (formatter + .editorconfig on-save + git branch), **`edit`** (insert, replace, surround, undo, …), **`visual`** (visual-mode helpers), **`comment`** (`<leader>/` comment toggle), **`multi_cursor`** (multi-cursor edits), **`dispatch`** (`apply_action` + operator/motion glue), **`input`** (per-mode key/mouse handlers + `:`-command dispatch), **`lsp_glue`** (LSP event handling + request helpers), **`dap_glue`** (DAP event handling + breakpoint/step requests), **`git_glue`** (gutter diff refresh + hunk navigation), **`copilot`** (GitHub Copilot LSP integration — ghost-text suggestions), **`picker_glue`** (generic picker open/handle + yazi), **`package_glue`** (`<leader>p` package-manager flow — manifest→package/search→version pickers over background threads), **`quickfix`** (quickfix/location-list state + navigation), **`windows`** (window-split focus + lifecycle around `layout.rs`), **`installer`** (`:install` overlay — three-stage state machine over `binvim::install`), **`health`** (`:health` output).
- **`parser.rs`** turns raw `KeyEvent`s into `Action` values via a Vim-grammar state machine. Operators (`d`, `c`, `y`, …), motion verbs, text-object verbs, counts, registers, and pending-prefix state (`g`, `z`, `[`, `]`, leader, surround) are all resolved here. `app.rs` only sees the resolved `Action`.
- **`buffer.rs`** wraps `ropey::Rope`. It exposes byte/char/line conversions and a monotonically incrementing `version` field used as a cache key by `lang.rs` and the LSP debounce.
- **`motion.rs`** + **`text_object.rs`** are pure functions over `(buffer, cursor)` returning `MotionResult` / `TextRange`. They have the densest test coverage in the project.
- **`mode.rs`**, **`cursor.rs`**, **`undo.rs`** — small data-model modules. `undo.rs` also handles persistence to `~/.cache/binvim/undo/<sha>.json`, keyed by content hash so an external edit invalidates stale history.
- **`window.rs`** + **`layout.rs`** carry the window-split system. A `Window` is a view onto a buffer (cursor, viewport, visual anchor, buffer index); the active one lives inline on `App.window`, the rest in `App.windows: HashMap<WindowId, Window>`. `Layout` is the binary split tree whose leaves are `WindowId`s — `partition()` walks it against a parent `Rect` and emits `(WindowId, Rect)` per leaf, and `focus_neighbor()` uses those rects for **geometric** `h`/`j`/`k`/`l` navigation, not tree-order.
- **`session.rs`** persists the open-buffer set + cursor positions on clean shutdown to `~/.cache/binvim/sessions/<hash-of-cwd>.json`. Restored on launch **only** if no explicit file arg was passed — `binvim foo.rs` always means "open foo.rs."

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bgunnarsson/binvim](https://github.com/bgunnarsson/binvim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
