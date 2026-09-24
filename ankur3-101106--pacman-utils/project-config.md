---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

archman — an interactive TUI system manager for Arch Linux (pacman + AUR). A native **Rust** application built on **ratatui** + **crossterm**, styled after LinUtil: a two-pane browser (category sidebar → flat action list), description pane, ASCII logo header, per-category accent colors, and a `?` cheatsheet overlay. All styling flows through the helpers in `widgets.rs`, which also honor `NO_COLOR`. The original bash implementation was removed in v2; its sources remain reachable in git history.

The user-facing surface is declared entirely in `screens/registry.rs`: categories (`CatDef`) hold flat lists of actions (`ActionDef`), and each action either opens a screen (`Launch::Screen(fn(&App) -> Box<dyn Screen>)`) or runs immediately behind an optional confirmation (`Launch::Run(RunSpec)` with tag/confirm/danger/build/ok_msg/fail_msg/done_log). Adding a feature = one row there plus its module; nothing else needs touching.

## Commands

```bash
cargo build --release        # release binary at target/release/archman
cargo check                  # fast type-check (use before building)
cargo test                   # all unit tests
cargo test <name>            # single test, e.g. cargo test pushed_screen_clears_dashboard_pane
./target/release/archman     # run the TUI
./install.sh                 # build, then ask whether to install to /usr/local/bin
```

Tests live in `#[cfg(test)]` modules next to the code, five files total: `fuzzy.rs` (matcher), `settings.rs` (epoch/toggle round-trip), `pty.rs` (spawn/read/kill), and render regression tests using ratatui's `TestBackend` in `app.rs` and `screens/runpane.rs` (overlay bleed-through, toast placement). There is no clippy/rustfmt config or CI.

- CLI surface: `--version/-v`, `--help/-h`; anything else is the interactive TUI.
- The app mutates the system through `sudo pacman` etc. and requires a real terminal — never run it expecting non-interactive completion.
- Headless render testing works via ratatui's `TestBackend` (see the existing tests in `app.rs`/`runpane.rs` — build an `App`, `term.draw()`, assert on buffer text). For smoke-testing the *real* terminal path (raw mode, keys), drive the binary under a pty with an explicit winsize (e.g. Python `pty.fork()` + `TIOCSWINSZ`); a bare `script -qec` in CI-like shells reports a 0×0 window and renders nothing.

## Architecture

Single binary, one module per feature:

- `main.rs` — CLI flags, raw-mode/alternate-screen setup, panic hook that restores the terminal, `restore_terminal()`/`enter_tui()` used around external commands.
- `app.rs` — the core. Owns:
  - the **screen stack** (`Vec<Box<dyn Screen>>`, main menu always at index 0),
  - **modals** (`Confirm`/`Input`; empty input = cancel, mirroring v1's bash semantics),
  - toasts, and the queue of **external commands** (`ExtCmd`).
- `widgets.rs` — reusable pieces: header logo, help overlay, `Menu`, `FuzzyList`, `TextViewer`, `kv_table`, spinner. All styling goes through these helpers; screens never touch crossterm directly.
- `sys.rs` — every external interaction: pacman/AUR queries (`si`, `qi`, `-Slq`, orphans, explicit/foreign lists…), capability detection (`Caps::detect`, `has_bin`), mirror/cache/log parsing, and `Job<T>` (background thread + channel so slow/network queries don't freeze drawing).
- `screens/*` — one module per feature, each exposing small single-purpose screens implementing the `Screen` trait (`handle_key`/`poll`/`draw`/`busy`/`on_confirm`/`on_input`/`on_ext_done`). Shared helpers live in `screens/mod.rs` (`info_lines`, `args`). Query-style screens (OwnerQueryScreen, InfoQueryScreen, ImportScreen) open their input modal from `poll()` on the first tick via an `asked` flag — constructors only get `&App`.

### Conventions that span files

- **Flat state, not data enums.** Screens keep a `mode: Mode` discriminant (Copy enum) plus sibling fields (`pick: FuzzyList`, `detail: Detail`, …). Never write the matched field inside its own match arm (`self.state = …` while matching on `&mut self.state`) — it won't borrow-check; assign sibling fields instead.
- **The dispatch pitfall.** `App::dispatch` pops the top screen, runs the closure, then re-inserts it *at its original depth* so screens pushed during handling end up above it, and skipped entirely if the screen called `app.pop()`. If you change this dance, the symptom of getting it wrong is "navigation pushes a screen but the UI keeps drawing the old one".

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ankur3-101106/pacman-utils](https://github.com/ankur3-101106/pacman-utils) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
