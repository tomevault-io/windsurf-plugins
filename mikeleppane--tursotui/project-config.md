---
trigger: always_on
description: cargo build                  # build (never release mode during dev)
---

# tursotui — Terminal UI for Turso/SQLite

## Quick Reference

    cargo build                  # build (never release mode during dev)
    cargo run -- test.db         # run with a database file
    cargo run                    # run with :memory: database
    cargo test                   # run tests
    cargo fmt                    # format
    cargo clippy --all-features --all-targets -- -D warnings  # lint

## Architecture

See `docs/specs/design.md` for the full design spec.
Milestone plans live in `docs/plans/`.

### Key patterns

- **Component trait** (`src/components/mod.rs`): each panel implements `handle_key`, `update`, `render`
- **Action enum** (`src/app.rs`): all state mutations flow through actions (unidirectional data flow)
- **Two-phase dispatch**: `AppState::update()` handles state changes, then `dispatch_action_to_components()` in main.rs routes to components and I/O
- **DatabaseHandle** (`src/db.rs`): stores `Arc<Database>`, creates fresh connections per query task via `tokio::spawn`
- **Event loop** (`src/main.rs`): drain async channel → poll crossterm (16ms) → route to focused component → fallback to global keys → render
- **UiPanels** (`src/main.rs`): groups component instances
- **Status bar** (`src/components/status_bar.rs`): a render function, NOT a Component — no key handling, reads from AppState
- **Panel block helpers** (`src/components/mod.rs`): `panel_block()` and `overlay_block()` produce consistently-styled `Block` widgets with rounded borders and padded titles — all components use these instead of building blocks directly
- **Data editor** (`src/components/data_editor.rs`): manages ChangeLog (one-entry-per-PK invariant), DML generation, FK navigation stack, and EditRenderState for visual overlay injection into ResultsTable
- **Theme system** (`src/theme.rs`): Catppuccin Mocha (dark) and Latte (light) with semantic color roles — all colors referenced via Theme fields, never hardcoded

### Conventions

- `pub(crate)` visibility everywhere (no `pub` exports)
- `#[forbid(unsafe_code)]` — no unsafe allowed
- Pedantic clippy with selective allows — see `[lints.clippy]` in Cargo.toml
- All string width calculations must use `unicode-width`, never `.len()` (byte count)
- All panel borders use `BorderType::Rounded` via the `panel_block`/`overlay_block` helpers — never create `Block::bordered()` directly
- SQL identifiers must be escaped with `quote_identifier()` (double-quotes), literals with `quote_literal()` (single-quotes) — both in `data_editor.rs`
- Status bar is intentionally minimal — show panel name + context info + "F1 Help", no keybinding cheat sheets (those belong in the help overlay)
- **Overlay pattern**: `Overlay` enum must derive `Copy` — use unit variants only (no data). Store overlay state as a separate `Option<State>` field on `AppState`. Wire key routing in `input.rs`, rendering in `layout.rs`, action handling in `AppState::update()`.
- **Shift+Letter in standard terminals**: `Shift+S` sends `KeyCode::Char('S')` with `KeyModifiers::NONE`, NOT `KeyModifiers::SHIFT`. The SHIFT modifier is only populated under kitty keyboard protocol. Always match both: `(KeyModifiers::NONE | KeyModifiers::SHIFT, KeyCode::Char('S'))`.
- **Tab key priority**: When multiple subsystems compete for Tab (autocomplete, param bar, indentation), check in priority order BEFORE the autocomplete popup intercept. Autocomplete always has a selected candidate, so Tab would be swallowed otherwise.

### Turso/libsql compatibility

The authoritative reference is [COMPAT.md](https://github.com/tursodatabase/turso/blob/main/COMPAT.md).
Turso aims for full SQLite compatibility but has gaps that directly affect this project.

#### PRAGMA limitations

- **`foreign_key_list` NOT supported** — returns "Not a valid pragma name". FK info must be parsed from CREATE TABLE SQL in `SchemaEntry::sql` using `parse_foreign_keys()` in `db.rs`.
- **`defer_foreign_keys` NOT supported** — returns "Not a valid pragma name". Intentionally omitted from turso's pragma whitelist (`parser/src/ast.rs PragmaName` enum). Turso does support `DEFERRABLE INITIALLY DEFERRED` on column constraints and `BEGIN DEFERRED` transactions, but not the connection-level pragma. Our `execute_transaction` in `ops.rs` no longer calls this pragma — FK checks run immediately per-statement, so DML generation must order statements to respect FK dependencies.
- **Syntax quirk**: turso uses single-quoted values in PRAGMA SET (`PRAGMA name = 'value'`), not double-quoted.
- **65+ PRAGMAs work**, including: `foreign_keys`, `journal_mode`, `cache_size`, `page_size`, `table_info`, `index_info`, `integrity_check`, `quick_check`, `user_version`, `busy_timeout`.
- **`synchronous`** only supports `OFF` and `FULL` (not `NORMAL`).
- **Journal mode** restricted to WAL and MVCC only — rollback modes (`DELETE`, `TRUNCATE`, `PERSIST`, `MEMORY`) are NOT supported.
- **Notable unsupported PRAGMAs**: `auto_vacuum`, `mmap_size`, `locking_mode`, `optimize`, `secure_delete`, `recursive_triggers`, `collation_list`, `compile_options`, `threads`.
- **Turso-specific PRAGMAs**: `capture_data_changes_conn` (CDC), `cipher`/`hexkey` (encryption, experimental), `list_types` (custom type inspection).

#### SQL feature gaps


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mikeleppane/tursotui](https://github.com/mikeleppane/tursotui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
