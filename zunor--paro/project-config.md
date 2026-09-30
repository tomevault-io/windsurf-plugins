---
trigger: always_on
description: enables `parking_lot`, `paro-common` and `paro-journal`; do not describe the
---

# Paro contributor guide

This is repository knowledge for humans and coding agents, not configuration
for a particular assistant. Shared skills live in `.agents/skills/`; private
notes belong in the ignored `.agents/local/`. Keep this guide and shared skills
versioned. Classify a file's content and owner before deleting it: a tool-named
directory can also contain the only copy of architecture or workflow knowledge.

## Start with the selected checkout

- Inspect the requested worktree's HEAD, status and relevant diffs. Preserve
  unrelated staged, unstaged and untracked changes; do not switch to a fixed
  main checkout, reset user work or create extra large worktrees automatically.
- Read the affected crate's `lib.rs`, `Cargo.toml` and tests before changing its
  public boundary. Prefer the existing owner/contract over another parallel API.
- Use the toolchain in [rust-toolchain.toml](rust-toolchain.toml) and dependency
  versions in [Cargo.lock](Cargo.lock). Do not silently upgrade either to make a
  command pass. Actual flags and side effects come from the selected Makefile
  and CLI, not historical command lists.

## Architecture and ownership

Paro is a Rust columnar database combining relational, vector, full-text and
graph execution. `parod` is the PostgreSQL-wire front end. The query lifecycle
is approximately:

```text
server / session -> parser: SQL to AST
                 -> compiler: planner / binder -> optimizer -> compiled program
                 -> execution: admission -> selected image -> pipelines -> results
instance: database registry, shared resources, recovery and lifecycle
context: statement environment, resource accounting and cancellation
```

This is runtime flow, not a Cargo dependency graph. The compiler entry points
`compile_statement` and `compile_statement_with_parameter_types` consume an
already parsed AST; session owns parsing, prepared statements and cache policy.
A ready physical plan need not have materialized its execution image.
Admission and deferred lowering remain real work, not free work outside the
compiler timer. See [compiler/compile.rs](crates/compiler/src/compile.rs) and
the [optimizer contracts](crates/optimizer/readme.md).

The workspace members and feature-dependent edges are defined by the
[workspace manifest](Cargo.toml) and each crate's manifest. Update this map
when a crate is added, merged or moved; do not infer new dependencies from the
order of these rows.

| Crate | Responsibility / source entry |
| --- | --- |
| `paro-common` | [Shared types, errors, vectors/chunks, memory and configuration](crates/common/src/lib.rs) |
| `paro-parser` | [Tokenizer, SQL AST with source spans, parser and visitors](crates/parser/src/lib.rs) |
| `paro-planner` | [Binding, expressions, logical plans and shared immutable physical-plan contracts](crates/planner/src/lib.rs) |
| `paro-optimizer` | [Ordered rewrites, estimation, bounded regions, costing and physical construction](crates/optimizer/readme.md) |
| `paro-compiler` | [Planner/optimizer/execution orchestration](crates/compiler/src/lib.rs) |
| `paro-execution` | [Physical operators, expression evaluation, pipelines, spill and admission](crates/execution/src/lib.rs) |
| `paro-context` | [Statement/session environment, resources, write guards and cancellation](crates/context/src/lib.rs) |
| `paro-catalog` | [Schemas, catalog entries, MVCC and dependency tracking](crates/catalog/src/lib.rs) |
| `paro-function` | [Built-in function infrastructure](crates/function/src/lib.rs) |
| `paro-external` | [External ABI, routines, sources and worker runtime](crates/external/src/lib.rs) |
| `paro-storage` | [Buffer pool, rowsets/tablets, codecs, indexes, statistics and row storage](crates/storage/src/lib.rs) |
| `paro-journal` | [Ordered durable logging and publication](crates/journal/src/lib.rs) |
| `paro-transaction` | [Transaction types, snapshots, validation, locks and commit coordination](crates/transaction/src/lib.rs) |
| `paro-scheduler` | [Execution tasks, events, worker scheduling and coordination](crates/scheduler/src/lib.rs) |
| `paro-instance` | [Database registry, storage ownership, startup/recovery and shutdown](crates/instance/src/lib.rs) |
| `paro-session` | [SQL dispatch, prepared/portal state, transactions, COPY and results](crates/session/src/lib.rs) |
| `paro-server` | [pgwire connections/cancellation and the parod entry point](crates/server/src/bin/parod.rs) |

Dependency and implementation constraints:

- `paro-execution` consumes `paro-planner::physical`, not the optimizer.
  Planner must not depend on optimizer or execution. Test-only construction
  fixtures may depend on optimizer; production and target-specific dependency
  edges are checked by `tools/ci/check_plan_boundaries.py`.
- Lower-level storage/transaction facilities must not gain dependencies on
  planner, optimizer, execution or session merely to reuse a convenience type.
  Check normal versus dev dependencies and active features, not just a path.
- Preserve `paro-transaction`'s scalar-only boundary: its
  [manifest](crates/transaction/Cargo.toml) has a `types-only` configuration
  selected without default features. The default `runtime` feature currently

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zunor/paro](https://github.com/zunor/paro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
