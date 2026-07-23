---
trigger: always_on
description: - This repo hosts the **Prisma Engines**: PSL (schema parser/validator), schema-engine (migrate, introspect), query components (query compiler, driver adapters, compatibility harnesses), and utilities shared with Prisma Client.
---

# Agent Playbook — Prisma Engines

## 1. Big Picture
- This repo hosts the **Prisma Engines**: PSL (schema parser/validator), schema-engine (migrate, introspect), query components (query compiler, driver adapters, compatibility harnesses), and utilities shared with Prisma Client.
- Prisma 7 roadmap status:
  - `url`, `directUrl` and `shadowDatabaseUrl` are **invalid** in PSL.
  - CLI/tests override connection info via schema-engine CLI (`--datasource`) or shared `TestApi::new_engine_with_connection_strings`.
  - Reference commit: `34b5a692b7bd79939a9a2c3ef97d816e749cda2f` (driver adapter override plumbing).
- Prisma has removed the **native Rust query engine** in favor of the **Query Compiler (QC)** architecture:
  - Query planning happens in Rust (`query-compiler` crate). Output: an expression tree (“query plan”).
  - Query interpretation/execution runs in Prisma Client TypeScript using driver adapters. The interpreter has no knowledge of connection strings or even whether it talks to a real DB.
  - A compatibility harness (`qc-test-runner.ts` in the main repo) still emulates the legacy GraphQL protocol so the CLI and tests behave as before while consumers migrate.
  - MongoDB support is not yet implemented for QC; Prisma 7 will ship without MongoDB, to be added later once a driver adapter exists.

---

## 2. Repository Orientation
Key directories:
- `psl/` – Prisma Schema Language parser, validator, config tooling.
- `schema-engine/` – Migration/introspection engine plus test suites.
- `prisma-fmt/` – Language server & formatter entry point (tests rely on `expect!` snapshots).
- `schema-engine/sql-migration-tests` / `sql-introspection-tests` – Heavy integration suites (require DBs).
- `query-engine/` – Unused MongoDB connector crate left for future reference and the connector test kit (integration tests exercise QC through the driver adapter executor here).
- `query-compiler/` – Query planner + associated Wasm, playground, and the new `core-tests` crate.
- `libs/` – Shared libraries (value types, driver adapters, test setup).
- `driver-adapters/` – Rust-side adapter utilities for the new query interpreter.

Supporting infra:
- Tests use Rust `cargo test`. Some suites expect database URLs in env (see §5).
- `test-setup` crate provisions databases when env vars are defined (Docker-based in CI).
- `UPDATE_EXPECT=1 cargo test …` regenerates `expect!` snapshots (common when diagnostics shift).

---

## 3. Current Domain Knowledge
### Datasource URLs
- PSL rejects `directUrl`/`shadowDatabaseUrl` with targeted diagnostics (`DatamodelError::new_datasource_*_removed_error`).
- Parser still records `url` (and uses span for override fallbacks).
- `Datasource::override_urls()` now fakes spans because overrides bypass PSL parsing.
- Schema-engine tests must supply overrides via `TestApi::new_engine_with_connection_strings(connection_string, Some(shadow_connection))`. The wrapper returns an `EngineTestApi`.
- Old fixtures relying on `directUrl` inside PSL must be rewritten or deleted.
- Query compiler already assumes datasource URLs are supplied externally (from Prisma Client).

### Text Completions
- `prisma-fmt` completions now only offer `url` (no more direct/shadow suggestions).
- Completion scenarios removed for the deprecated properties. Expect JSON fixtures to change if docs/completions change again.

### Diagnostics / Tests
- Many tests assert on colored output via `expect!`. Always regenerate expectations when diagnostics wording changes.
- Integration tests around multi-schema migrations still need real DB URLs; without them they skip/fail early.
- Query compiler tests use insta snapshots (`query-compiler/tests`). Regenerate with `UPDATE_EXPECT=1 cargo test -p query-compiler`.
- The connector test kit (`query-engine-tests`) relies on `cargo insta` snapshots too (see `connector-test-kit-rs` README).

---

## 4. Typical Workflows
### Linting / Formatting
- Rustfmt + cargo fmt (standard). JSON fixtures kept raw (no formatter).
- Full lint pass (formatting + clippy warnings as errors):
  ```bash
  make pedantic
  ```
  This runs `cargo fmt -- --check` and `cargo clippy --all-features --all-targets -Dwarnings`. Fix the compiler/clippy diagnostics first, then formatting.

### Running Tests
1. **Fast PSL/LSP suites**
   ```bash
   cargo test -p prisma-fmt -F psl/all
   ```
   Use `UPDATE_EXPECT=1` to refresh snapshots.

2. **Unit tests in PSL**
   ```bash
   cargo test -p psl -F all
   ```

3. **Unit tests for the whole workspace**
  ```bash
  make test-unit
  ```
  Use this one if you can't figure out the correct cargo features for a specific crate.
  Some library crates may be tricky to compile in isolation without feature unification.
  Unit tests are very fast so there's no problem running them for the whole workspace.

4. **Schema engine SQL tests**
   Require DB env vars (see `.test_database_urls/` in repo root). Example:
   ```bash
   source .test_database_urls/postgres
   cargo test -p sql-migration-tests migration_with_shadow_database -- --nocapture
   ```

5. **Schema engine integration**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prisma/prisma-engines](https://github.com/prisma/prisma-engines) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
