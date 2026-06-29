---
trigger: always_on
description: Orientation for AI agents (and humans) working on **ttop**.
---

# AGENTS.md

Orientation for AI agents (and humans) working on **ttop**.

> Keep this file up to date **in the same change** whenever the architecture,
> module responsibilities, data flow, keybindings, or workflow change.

## What ttop is

A small terminal UI (TUI) that lists the processes running inside every **tmux**
pane, grouped by session and window — like `htop`, scoped to tmux. You can
navigate, filter, sort by CPU/memory, and kill processes.

It is read-only against the tmux server (it never creates/kills tmux objects);
the only side effect is sending signals to processes the user selects.

## Tech stack

- Rust, **edition 2024** (requires rustc ≥ 1.95; uses `let`-chains).
- [`ratatui`](https://crates.io/crates/ratatui) 0.30 for the TUI, driven through
  its re-exported `ratatui::crossterm`. Terminal lifecycle via
  `ratatui::init()` / `ratatui::restore()`.
- [`sysinfo`](https://crates.io/crates/sysinfo) 0.39 (`default-features = false`,
  `features = ["system"]`) for per-process CPU%, memory and parent PID, and for
  sending signals (`Process::kill_with`).

## Architecture

Data flows once per ~2s tick: **tmux structure + sysinfo processes → model tree
→ app rows → ui render**.

| File | Responsibility |
| --- | --- |
| `src/main.rs` | Entry point, terminal setup/teardown, event loop. `TICK = 2000ms`: redraw, poll a key (`event::poll`/`read`, Press only), dispatch to `App::on_key`, and `App::refresh()` every tick. `enable_enhanced_keys()` pushes `DISAMBIGUATE_ESCAPE_CODES` (when supported) so Shift+Enter is distinguishable from Enter. |
| `src/tmux.rs` | Shell out to `tmux` (`display-message`, `list-sessions`, `list-panes -a`). `query() -> TmuxInfo`. Structs: `Pane`, `TmuxInfo` (panes, session order, current session). Tab-separated `-F` format parsing. |
| `src/model.rs` | `Collector` owns the `sysinfo::System` (so CPU% is computed across refreshes). `build_sessions(&Collector, &TmuxInfo) -> Vec<Session>`. Structs: `Proc`, `Window`, `Session`. Each process is attached to the window of its **nearest pane-root ancestor** (walk parent chain until a `pane_pid` is hit). `command_string` joins argv, falling back to the process name. |
| `src/app.rs` | All state and logic. `App` (sessions, selected tab, sort, collapsed set, filter, selection, status), `Sort` (`Cpu`/`Mem`), `Row` (`Window`/`Proc`, where `Proc` carries a rendered tree `prefix`). `rows()` builds, per window, a parent→child process **tree** (`cmp_proc` sorts siblings, `emit_tree` draws the `├─`/`└─`/`│` branches), then orders the windows themselves by their total usage (`cmp_usage`, busiest first) before flattening to visible rows. `on_key` holds the **keybinding map**. Actions: tab switch, fold, filter input, `kill_selected`. |
| `src/ui.rs` | Pure rendering: `draw(&mut Frame, &App)`. Borderless, with a small margin: tabs (current session marked `●`, pre-selected), a column header + the tree `List`, and a footer (help / filter input / transient status). Colors come from the `DIM`/`DIM_SEL`/`SELECT_BG` constants plus `cpu_style`/`mem_style`; helpers `format_bytes`, `truncate`. |

### Conventions in this codebase

- The selection model is index-based into the `Vec<Row>` returned by
  `App::rows()`; `clamp()` keeps it in range after any change. `rows()` is the
  single source of truth shared by navigation, actions, and rendering.
- Sorting, filtering, collapsing and **tree building** live in `rows()`, not in
  `model`. The model emits a flat, unsorted list of processes (with `ppid`);
  `rows()` reconstructs the per-window tree. Filtering keeps matches **and their
  ancestors** so a match is still shown within its branch.
- Comment sparingly — only where intent isn't obvious (see existing doc comments).

## Keybindings (also shown in the footer)

| Key | Action |
| --- | --- |
| `j` / `k`, `↓` / `↑` | Move selection |
| `Ctrl-d` / `Ctrl-u` | Half-page down / up |
| `PgDn` / `PgUp` | Page down / up |
| `h` / `l`, `←` / `→`, `Tab` / `Shift-Tab` | Switch session tab |
| `g` / `G`, `Home` / `End` | Jump to top / bottom |
| `Enter` / `Space` | Fold / unfold a window |
| `Shift-Enter` | Fold / unfold **all** windows |
| `/` | Filter (command or PID); `Enter` applies, `Esc` clears |
| `P` / `p` | Sort by CPU |
| `M` / `m` | Sort by memory |
| `t` | `SIGTERM` the selected process |
| `x` | `SIGKILL` the selected process |
| `q` / `Ctrl-C` | Quit |

Navigation is vim-style (`hjkl`, `g`/`G`, `Ctrl-d`/`Ctrl-u`); arrows/Home/End/Page
also work. Page scrolling uses the list height cached in `App::viewport` (set by
the renderer). `P`/`M` follow `top` conventions. Kills moved off `k` (now "up")
to `t`/`x`.

## Build / lint / run

```sh
cargo build
cargo clippy --all-targets   # keep clean (no warnings)
cargo fmt                    # or: cargo fmt --check
cargo run --release          # run it (from inside tmux to highlight current session)
```

## Verifying changes (important)

This repo has no committed tests, and **the interactive `tuistory` skill cannot
be used here** — the dev environment itself runs inside a tuistory session, so
its isolation wrapper deadlocks. Verify headlessly instead:

- Render the real `App` to `ratatui::backend::TestBackend` and assert on the
  buffer (`terminal.backend().to_string()`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [caarlos0/ttop](https://github.com/caarlos0/ttop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
