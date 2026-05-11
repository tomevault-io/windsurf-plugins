---
trigger: always_on
description: * `crates/icydb`: Public meta-crate re-exporting the workspace API.
---

# Repository Guidelines

## Project Structure & Module Organization

* `crates/icydb`: Public meta-crate re-exporting the workspace API.
* `crates/icydb-core`: Runtime, storage, executors, and core types.
* `crates/icydb-schema-derive`: Derive and codegen macros.
* `crates/icydb-schema`: Schema AST/builders and validation.
* `crates/icydb-build`: Build/codegen helpers and canister glue.
* `canisters/audit/minimal`: Minimal SQL canister harness used for wasm audit baseline.
* `canisters/audit/one_simple`: One-entity simple SQL audit harness.
* `canisters/audit/one_complex`: One-entity complex SQL audit harness.
* `canisters/audit/ten_simple`: Ten-entity simple SQL audit harness.
* `canisters/audit/ten_complex`: Ten-entity complex SQL audit harness.
* `canisters/demo/rpg`: Character-only RPG demo/perf canister harness.
* `canisters/test/sql_parity`: Broad SQL parity/explain/perf test canister harness.
* `canisters/test/sql`: Lightweight SQL smoke-test canister harness.
* `schema/demo/rpg`: Character-only RPG demo schema fixtures and seed data.
* `schema/test/sql_parity`: Broad SQL parity test-canister schema fixtures.
* `schema/audit/minimal`: Minimal schema fixtures for lightweight wasm auditing.
* `schema/audit/one_simple`: One-entity simple SQL audit schema fixtures.
* `schema/audit/one_complex`: One-entity complex SQL audit schema fixtures.
* `schema/audit/ten_simple`: Ten-entity simple SQL audit schema fixtures.
* `schema/audit/ten_complex`: Ten-entity complex SQL audit schema fixtures.
* `schema/test/fixtures`: Shared schema fixtures for macro/e2e test harnesses.
* `schema/test/sql`: Lightweight SQL smoke-test schema fixtures.
* `testing/macro-tests`: Macro and schema contract tests.
* `testing/pocket-ic`: Pocket-IC integration tests.
* `assets/`: Images and docs assets. `scripts/`: release/version helpers. `Makefile`: common tasks.
* Workspace manifest: `Cargo.toml` (edition 2024, rust-version 1.95.0).

---

## Build, Test, and Development Commands

* `make check`: Fast type-check for all crates.
* `make test`: Run all unit/integration tests (`cargo test --workspace`).
* `make build`: Release build for the workspace.
* `make clippy`: Lints with warnings denied.
* `make fmt` / `make fmt-check`: Format or verify formatting.
* Versioning: `make version|tags|patch|minor|major|release`.

---

## Common Workflows

* Pre-commit gate (local): `make fmt-check && make clippy && make check && make test`.
* Fast CI gate (local): `make check && make clippy`.
* Release (local): `make security-check && make release`.

---

## Language Policy

* Do not add Python code to this repository.
* New tooling, scripts, test helpers, and build helpers must use the repo’s existing Rust/shell patterns instead of introducing Python.

---

## Wasm Measurement Priority

* For wasm optimization decisions, treat non-gzipped wasm bytes as the primary metric.
* Primary decision metrics are:
  * built `.wasm`
  * shrunk `.wasm`
* Deterministic gzip artifacts (`.wasm.gz`) are secondary transport metrics only.
* Do not reject or accept a wasm change primarily because of gzip movement when raw non-gzipped wasm improves.
* Mention gzip deltas only as supporting context or when they are unexpectedly large enough to warrant a note.

---

## Concurrent Editing During Agent Runs

* User edits made while the coding agent is running are expected in this repository.
* Mid-run file changes should be treated as normal collaboration, not an automatic stop condition.
* When concurrent edits are detected, the agent should re-read the affected files and continue unless there is a real conflict on the same logic block.
* Do not treat unrelated concurrent file changes as an error state.

---

## Agent File Links

* When referencing files in agent responses, always use absolute filesystem paths so links are clickable in VS Code.
* Relative paths are allowed in code and commands, but response links must be absolute.
* Example: `/home/adam/projects/icydb/crates/icydb-core/src/db/session.rs:53`

---

## Git Hooks

* Hooks path: `.githooks` (auto-configured via `core.hooksPath`).
* Pre-commit runs: `cargo fmt --all -- --check`, `cargo sort --check`, `cargo sort-derives --check`.
* Auto-setup: running common Make targets (`fmt`, `fmt-check`, `clippy`, `check`, `test`, `build`, `install-dev`) ensures hooks are enabled.
* Tools: install with `make install-dev` (installs `cargo-sort` and `cargo-sort-derives`).

---

## Imports & Module Boundaries

Imports are considered part of a module’s public shape and architectural contract.

### Required

* All module import declarations MUST appear at the top of the file, immediately after module-level doc comments (if any).
* Required top-of-file order is strict: first `mod ...;` declarations, then one blank line, then `use ...;` imports, then one blank line, then `pub use ...;` re-exports.
* This strict import/declaration order applies to all files, including test files.
* All non-test modules MUST declare imports at the top of the file.
* Prefer a single top-level `use crate::{ ... };` block per module.
* For `prelude` imports, never name individual items. Use `prelude::*` or do not import the prelude at all.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dragginzgame/icydb](https://github.com/dragginzgame/icydb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
