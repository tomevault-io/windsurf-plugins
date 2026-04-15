---
trigger: always_on
description: - This workspace is a Rust 2024 workspace with two crates: the main `tideorm` crate and the `tideorm-macros` proc-macro crate.
---

# TideORM Workspace Instructions

## Scope

- This workspace is a Rust 2024 workspace with two crates: the main `tideorm` crate and the `tideorm-macros` proc-macro crate.
- Prefer minimal, targeted edits. Do not reformat unrelated Rust code or documentation.
- Keep public APIs consistent unless the task explicitly requires a breaking change.

## Core Commands

- Fast smoke test: `cargo test --lib`
- Broad unit compatibility pass: `cargo test --all-features --lib`
- Proc-macro pass: `cargo test -p tideorm-macros --lib`
- Default backend test pass: `cargo test --features postgres`
- SQLite integration suite: `cargo test --test sqlite_integration_tests --features "sqlite runtime-tokio" --no-default-features`
- PostgreSQL integration suite: `cargo test --test postgres_integration_tests`
- Advanced PostgreSQL suite: `cargo test --test postgres_advanced_tests`
- MySQL integration suite: `cargo test --test mysql_integration_tests --features mysql`
- Strict lint pass: `cargo clippy --all-targets --all-features -- -D warnings`
- Benchmarks: `cargo bench`
- Docs build: `mdbook build`
- Docs preview: `mdbook serve --open`

## Environment And Prerequisites

- Rust `1.85+` is required.
- Integration tests may require database servers and environment variables loaded from `.env` via `dotenvy`.
- PostgreSQL tests default to `postgres://postgres:postgres@localhost:5432/test_tide_orm` when env vars are absent.
- SQLite tests default to `sqlite://./test_tide_orm.db?mode=rwc` when `SQLITE_DATABASE_URL` is absent.
- Skip flags are supported: `SKIP_SQLITE_TESTS`, `SKIP_MYSQL_TESTS`, `SKIP_POSTGRES_TESTS`.
- Benchmarks expect PostgreSQL access through `POSTGRESQL_DATABASE_URL`.

## Architecture Boundaries

- `src/` contains the public ORM runtime. `tideorm-macros/src/` contains proc-macro parsing and code generation.
- Keep SeaORM as an internal implementation detail. The main boundary is centered in `src/internal/`; do not expose SeaORM types in the public API unless the task explicitly requires it.
- `src/database/` contains the global connection pattern, scoped connection overrides, and transaction helpers used by the rest of the crate.
- `src/model/` contains generated-model runtime support, CRUD internals, nested save behavior, and serialization helpers that rebuild runtime relation wrappers after serde rewrites.
- `src/query/` contains query-builder behavior, SQL generation, and DB-specific query helpers. The main SQL seams are `src/query/sql/`, `src/query/db_sql.rs`, and `src/query/structure/`.
- `src/internal/backend.rs` and `src/internal/sql_safety.rs` are shared boundaries for backend-specific statement construction, identifier quoting, and raw-SQL safety.
- `src/sync/schema.rs` contains TideORM schema-sync type mapping and table creation helpers.
- `tideorm-macros/src/parse.rs` and `tideorm-macros/src/parse/` contain proc-macro attribute parsing, validation parsing, and index parsing.
- Optional modules and feature-gated APIs are public-surface only: `attachments`, `translations`, `fulltext`, `entity-manager`, and dirty-tracking-backed model inspection helpers do not exist unless their feature is enabled.

## Project Conventions

- Preserve feature gating with `#[cfg(feature = ...)]` on optional modules, optional APIs, and tests.
- Keep generated-model behavior aligned with the macros crate and runtime crate together; changes often require touching both crates.
- Prefer existing builder and helper layers over ad hoc SQL or duplicated logic.
- Maintain the global database initialization model built around `TideConfig::init()` rather than introducing alternate connection flows unless explicitly requested.
- When a non-test Rust file grows large, prefer extracting focused sibling modules instead of continuing to grow a monolith. Keep non-test helper `mod` declarations near the top of the file.
- Preserve runtime-only relation helper state across serialization-based model overwrites by keeping the `refresh_runtime_relations_from` flow intact.
- The crate denies unsafe code. Avoid introducing `unsafe`.

## Safety-Critical Rules

- Do not build SQL with interpolated identifiers or values. Use the shared safety boundary in `src/query/db_sql.rs`, parameterized SQL, or `Expr::cust_with_values`.
- When quoting identifiers, reuse the shared quoting helpers instead of manual escaping.
- Do not pass user-controlled full-text search input directly into PostgreSQL or SQLite full-text expressions. Reuse the sanitizers in `src/internal/sql_safety.rs`.
- Preserve the explicit-filter bulk-mutation guards in `src/query/sql/execution/mutation_safety.rs`; unfiltered destructive mutations should stay blocked unless the public API explicitly allows them.
- Preserve tokenization semantics: missing encryption configuration should surface as `Err`, while invalid or tampered tokens should remain `Ok(None)`.
- Callback dispatch must happen at concrete macro-generated call sites. Do not move callback execution into generic helpers that erase the concrete model type.

## Testing Guidance

- For quick validation after a library change, start with `cargo test --lib`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mohamadzoh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
