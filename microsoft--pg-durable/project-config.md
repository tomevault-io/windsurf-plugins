---
trigger: always_on
description: pg_durable is a **PostgreSQL extension** (pgrx/Rust) providing durable SQL function execution. Everything runs inside the PostgreSQL server—no external services.
---

# pg_durable AI Coding Instructions

## Architecture Overview

pg_durable is a **PostgreSQL extension** (pgrx/Rust) providing durable SQL function execution. Everything runs inside the PostgreSQL server—no external services.

**Two execution contexts:**
1. **Backend processes** (user sessions): Build function graphs via DSL operators (`~>`, `|=>`, `&`, `|`)
2. **Background worker**: Executes graphs durably via [duroxide](https://github.com/microsoft/duroxide) runtime

**Data flow:** User calls `df.start()` → nodes saved to `df.nodes` → instance queued → background worker picks up → duroxide orchestration executes nodes → results in `df.instances`

## Key Files & Modules

| Path | Purpose |
|------|---------|
| [src/lib.rs](../src/lib.rs) | Extension entry, schema/table definitions, SQL operators |
| [src/dsl.rs](../src/dsl.rs) | DSL functions: `df.sql()`, `df.seq()`, `df.if()`, `df.loop()` |
| [src/worker.rs](../src/worker.rs) | Background worker setup, duroxide runtime initialization, and duroxide schema migration (`ApplyAll`) |
| [src/orchestrations/](../src/orchestrations/) | Duroxide orchestrations (⚠️ deterministic code only) |
| [src/activities/](../src/activities/) | Duroxide activities (I/O happens here) |
| [src/types.rs](../src/types.rs) | Core types: `Durofut`, `FunctionGraph`, `FunctionNode` |
| [tests/e2e/sql/](../tests/e2e/sql/) | SQL-based E2E tests (numbered, run sequentially) |
| [sql/pg_durable--0.1.1.sql](../sql/pg_durable--0.1.1.sql) | First version install SQL fixture (for upgrade testing) |
## Development Commands

```bash
# Build extension
cargo build                    # or: make build

# Check formatting (run after every code change)
cargo fmt -p pg_durable -- --check

# Fix formatting
cargo fmt -p pg_durable

# Run unit tests (pgrx)
./scripts/test-unit.sh         # uses: cargo pgrx test pg17

# Run E2E tests locally
./scripts/test-e2e-local.sh              # all tests
./scripts/test-e2e-local.sh --verbose    # -v, shows test output, use with filtering
./scripts/test-e2e-local.sh 04_parallel  # filter/specific test
./scripts/test-e2e-local.sh --keep       # keep server running for debugging

# Run upgrade tests (schema comparison + backward compat)
./scripts/test-upgrade.sh                # test upgrade from previous version
./scripts/test-upgrade.sh --verbose      # show detailed diff on failure

# Connect to test database (after --keep)
~/.pgrx/17.*/pgrx-install/bin/psql -h localhost -p 28817 -d postgres

# View background worker logs, do this especially when debugging E2E tests
tail -f ~/.pgrx/17.log

# Stop test server
./scripts/pg-stop.sh
```

## Critical Patterns

### Orchestrations Must Be Deterministic
Files in `src/orchestrations/` must be 100% deterministic—no I/O, no `Utc::now()`, no random numbers. All side effects go through activities.

### Activity Naming Convention
Each activity has a co-located `NAME` constant for IDE navigation:
```rust
// src/activities/execute_sql.rs
pub const NAME: &str = "pg_durable::activity::execute-sql";
```

### DSL Creates Graph Nodes
DSL functions like `df.sql()` insert rows into `df.nodes`. The `Durofut` struct represents a node reference passed between operators.

### E2E Test Structure
Tests in `tests/e2e/sql/` follow this pattern:
1. Create temp state table, call `df.start()`
2. Poll `df.status()` in a loop until completed/failed
3. Assert results, raise exception on failure
4. Cleanup and output `SELECT 'TEST PASSED'`

### Binary Backward Compatibility
The new `.so` must work against **all** previous versions' schemas (same major version) because customers may never run `ALTER EXTENSION UPDATE`. When changing SQL queries in Rust code, ensure they work against both old and new schemas (see [docs/upgrade-testing.md](../docs/upgrade-testing.md)). CI enforces this via `scripts/test-upgrade.sh`.

## Common Tasks

**Adding a new DSL function:** Add to [src/dsl.rs](../src/dsl.rs) with `#[pg_extern(schema = "df")]`

**Adding a new activity:** Create file in `src/activities/`, add `pub const NAME`, register in [src/registry.rs](../src/registry.rs)

**Adding E2E test:** Create numbered SQL file in `tests/e2e/sql/`, follow existing pattern (see [02_sequence.sql](../tests/e2e/sql/02_sequence.sql))

**Changing the `df` extension schema:** If the upgrade script doesn't exist yet, follow the "Preparing for the next version" section in [docs/upgrade-testing.md](../docs/upgrade-testing.md). Then: add DDL to the upgrade script (`sql/pg_durable--<prev>--<current>.sql`), ensure `.so` backward compat with all previous schemas, and add a section to "Version-Specific Changes" in [docs/upgrade-testing.md](../docs/upgrade-testing.md). Note: duroxide schema changes do not require extension upgrade script changes — the BGW applies them automatically at startup.

For Scenario A, treat the upgrade path as the contract for already-shipped versions: before release, fresh install for the new version should match what an existing customer gets by installing the previous version and applying the upgrade chain.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/pg_durable](https://github.com/microsoft/pg_durable) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
