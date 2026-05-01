---
trigger: always_on
description: Guidance for AI coding agents (Claude Code, Codex, Cursor, etc.) working in
---

# AGENTS.md

Guidance for AI coding agents (Claude Code, Codex, Cursor, etc.) working in
this repository. Human contributors should read [CONTRIBUTING.md](CONTRIBUTING.md);
this file complements it with the architectural context an agent needs to
make safe, idiomatic changes.

## Project

cell is a terminal spreadsheet editor with Vim keybindings, written in Rust.
It supports CSV/TSV import/export and a native `.cell` format that preserves
formulas. It also ships a non-interactive headless mode for scripting.

## Build & test commands

```sh
cargo build                                 # build all crates
cargo build --release                       # release build (binary at target/release/cell)
cargo test                                  # all tests (unit + integration)
cargo test -p cell-sheet-core               # core library only
cargo test -p cell-sheet-tui                # TUI crate only
cargo test -p cell-sheet-core -- col_label  # single test by name
cargo fmt --all --check                     # check formatting (CI)
cargo clippy --workspace --all-targets --all-features
cargo install --path crates/cell-sheet-tui  # install the `cell` binary
```

CI runs `fmt`, `clippy`, `test`, and `build` on Linux, macOS, and Windows
with `RUSTFLAGS=-Dwarnings`. Any warning fails the build, so run clippy
locally before claiming work is complete.

## Architecture

Cargo workspace with two crates:

- **`cell-sheet-core`** — pure data library with **zero TUI dependencies**.
  Contains the data model (`Sheet`, `Cell`, `CellValue`, `CellPos`), formula
  engine (tokenizer → parser → AST → evaluator), dependency graph with
  topological recalculation, and file I/O (CSV/TSV via the `csv` crate,
  native `.cell` format).
- **`cell-sheet-tui`** — terminal UI built on `ratatui` + `crossterm`.
  Implements Vim modal editing (Normal, Insert, Visual, VisualBlock, Command),
  the main event loop, rendering, undo/redo, clipboard with formula-aware
  paste, viewport scrolling, and the headless CLI.

### Key data flow

1. User edits a cell → `Action::EditCell` dispatched to `App::process_action`.
2. If formula (`=` prefix): `deps::set_formula` parses it, registers it in
   `DepGraph`, then `mark_dirty` propagates to dependents, then `recalculate`
   does topological-sort evaluation.
3. If plain value: `Sheet::set_cell` auto-detects type (number vs text).

### Formula engine pipeline (`cell-sheet-core`)

`token.rs` (lexer) → `parser.rs` (recursive descent → `Expr` AST) →
`eval.rs` (tree-walk evaluator) → `functions.rs` (built-in function
dispatcher). The authoritative list of supported functions lives in
`functions.rs`; the user-facing list lives in
`help/entries.rs::FORMULA_ENTRIES`. These two must stay in sync — see the
Help system section below.

### Dependency graph (`deps.rs`)

`DepGraph` tracks bidirectional edges (dependencies ↔ dependents). On any
cell change, `mark_dirty` does BFS propagation, then `recalculate` does
Kahn's algorithm topological sort. Circular references produce `#CIRC!`
errors.

### Modes (`cell-sheet-tui`)

Each mode has its own input handler under `crates/cell-sheet-tui/src/mode/`:
`normal.rs` (with multi-key sequence support like `gg`, `dd`), `insert.rs`,
`visual.rs`, `command.rs` (`:` commands and `/` search), and `help.rs` (the
modal help viewer entered via `:help`). Add new key sequences in the
appropriate mode handler rather than threading them through `app.rs`.

### Help system (`cell-sheet-core::help`)

The in-app help screen and `:help <topic>` are data-driven. `HelpRegistry`
(in `crates/cell-sheet-core/src/help/mod.rs`) is built from the static
slices in `crates/cell-sheet-core/src/help/entries.rs`
(`NORMAL_ENTRIES`, `INSERT_ENTRIES`, `VISUAL_ENTRIES`, `COMMAND_ENTRIES`,
`FORMULA_ENTRIES`). The TUI's `render/help.rs` renders directly from that
registry — if an entry is missing, the help screen and `:help` will silently
omit it. Treat `entries.rs` as a first-class part of any user-visible
change, not as documentation that lags behind.

## Conventions

- `CellPos` is `(usize, usize)` = `(row, col)`, zero-indexed.
- Column labels are Excel-style (A, B, …, Z, AA, AB, …) — convert with
  `col_index_to_label` / `col_label_to_index`.
- Formulas always start with `=`. The `raw` field stores the original input;
  `value` stores the computed result.
- CSV export flattens formulas to computed values; the `.cell` format
  preserves them.
- The core crate must remain independent of any TUI dependency. New
  evaluation, parsing, or storage logic belongs in `cell-sheet-core`; new
  rendering, input, or terminal logic belongs in `cell-sheet-tui`.
- Every user-visible keybinding, `:` command, and built-in formula function
  must have a corresponding `HelpEntry` in
  `crates/cell-sheet-core/src/help/entries.rs`. The help renderer reads that
  registry as its single source of truth — if it isn't there, `:help` will
  lie to the user.

## Working in this repo as an agent

### Before changing code

- Read the relevant module in full before editing — files are small and the
  context is usually worth the tokens.
- For any change that touches behavior, identify where the existing tests
  live (`crates/<crate>/src/<module>.rs` typically has `#[cfg(test)]`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [garritfra/cell](https://github.com/garritfra/cell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
