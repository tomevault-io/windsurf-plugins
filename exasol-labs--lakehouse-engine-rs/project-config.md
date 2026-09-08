---
trigger: always_on
description: **Spec-driven project using speq-skill.**
---

# Project Rules

**Spec-driven project using speq-skill.**

Project mission in: @specs/mission.md

## Feature tracking

- **New features are tracked as GitHub issues** (`gh issue create`) before/at the start of
  work, in addition to speq spec deltas. Reference the issue in the implementing commit
  (`Closes #<n>`) so the work and its tracking stay linked.

## Code navigation & editing

- **Prefer Serena's MCP symbolic tools over `grep`/`Read`/`Edit` for any code file.**
  Use `get_symbols_overview` / `find_symbol` for discovery and `find_referencing_symbols`
  for usages; use `replace_symbol_body`, `insert_after_symbol`, `insert_before_symbol`,
  `rename_symbol`, or `safe_delete_symbol` for edits — never a raw `Edit` on a symbol
  you reached via Serena. `grep`/`Glob` remain fine for discovery only; `Read`/`Edit`
  remain fine for non-code files (docs, specs, config) or a file already fully read
  into context this session.
- If Serena's tools are not yet loaded this session, load them and call
  `initial_instructions` before the first code read/grep/edit — don't default to
  built-in tools out of habit.

## Unit test layout

- **No test code in a production source file.** Unit tests MUST live in a sibling file named after
  the module's own file, declared with the module's other `mod` declarations or as the last item of
  that module:
  ```rust
  #[cfg(test)]
  #[path = "<module>_tests.rs"]
  mod tests;
  ```
  `foo.rs` → `foo_tests.rs`, `lib.rs` → `lib_tests.rs`, `foo/mod.rs` → `foo/foo_tests.rs`
  (`mod_tests.rs` would be meaningless). `#[path]` resolves relative to the declaring file's own
  directory.
- The file name MUST match `[0-9a-zA-Z_-]+[_-]tests.rs`. `cargo llvm-cov` excludes exactly that
  pattern from every report; any other name silently re-inflates the coverage percentage.
- The test module remains a child module of its parent, so `use super::*;` still reaches the
  parent's private items and its imports.
- A test-only helper belongs in the sibling `_tests.rs` file, not in the production module — add it
  there as `impl super::TypeName { ... }` or a plain free fn, **not** gated by `#[cfg(test)]`, since
  the whole file is already test-only. A helper shared across several sibling `_tests.rs` files gets
  its own file, named to match the pattern and declared with `#[path]` so the `mod` identifier keeps
  its honest name: `#[path = "test_support_tests.rs"] mod test_support;`. Compile-time surface
  probes are test-only code and follow the same rule
  (`#[path = "scan_surface_probe_tests.rs"] mod scan_surface_probe;`).
- The only `#[cfg(test)]` that may remain in a production module is a re-export widening visibility
  for tests (`#[cfg(test)] pub use ...`) — it has to sit in the module owning the item. Test *code*
  never stays behind.

## PR title convention

PR titles MUST follow Conventional Commits format: `<type>(<scope>): <description>` (scope is
optional but recommended), using one of: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`,
`perf`. The title MUST describe the change's target/final state once implemented — not its
current lifecycle stage. A plan-only PR for a new feature is still `feat(...)`, not a "planning"
or "spec" prefix, even though only spec deltas are committed so far.

`lakehouse-engine` (repo `lakehouse-engine-rs`; `-rs` = built in Rust) — an in-place
lakehouse query engine: technically an Exasol Virtual Schema, but it runs the DataFusion engine on
the node, in place, for querying Iceberg / Databricks from Exasol SQL.

## Iceberg and Delta Lake specification compliance

Any feature planned via `/speq:plan` that touches scanning, pushdown, or schema/type handling MUST
be checked against the Apache Iceberg table spec (https://iceberg.apache.org/spec/) during
planning — quote the relevant normative section, don't rely on memory. A known deviation from the
spec must either be fixed in the same plan or recorded as an explicit, accurately-scoped tracked
exception — a GitHub issue cited inline in the spec (see the `(#27)` pattern in
`specs/datafusion-scan/scan-execution-field-id-projection/spec.md`); it must never be a silent gap.
A deviation driven by an Exasol target-type limitation (e.g. no struct/list/map types) is not a
gap for either the Iceberg or the Delta spec — but it must still be named as a deliberate
trade-off in the spec, not left unstated.

The same obligation applies to Delta: any feature planned via `/speq:plan` that touches Delta
scanning, pushdown, or schema/type handling MUST be checked against the Delta Lake protocol
(https://github.com/delta-io/delta/blob/master/PROTOCOL.md) during planning — quote the relevant normative section
(e.g. `§ Reader Requirements for Type Widening`), don't rely on memory. A known deviation from the
protocol must either be fixed in the same plan or recorded as an explicit, accurately-scoped
tracked exception — a GitHub issue cited inline in the spec, same convention as the Iceberg rule
above (see `specs/datafusion-scan/type-relaxation/spec.md` and
`specs/vs-adapter/delta-reader-feature-gating/spec.md` for the citation format); it must never be
a silent gap.

## Exasol / tooling

- Use Exasol Docker images to run Integration and E2E tests; they must **fail**, not skip, if no DB.
- Use `exapump` for all Exasol/BucketFS interaction.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [exasol-labs/lakehouse-engine-rs](https://github.com/exasol-labs/lakehouse-engine-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
