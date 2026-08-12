---
trigger: always_on
description: When in doubt about what a semantic view **means**, follow Snowflake: the DDL
---

# DuckDB Semantic Views — Project Instructions

## Snowflake is the model for *semantics*, DuckDB for *conventions*

When in doubt about what a semantic view **means**, follow Snowflake: the DDL
surface and clause set, what a metric / dimension / fact / relationship is, grain
and fan-trap behaviour, what each `SHOW` / `DESCRIBE` command reports. That is the
feature being built, and Snowflake is its reference implementation.

But Snowflake's *host-language* rules are Snowflake's, not ours. Where the two
disagree on something that belongs to the surrounding SQL dialect rather than to
semantic views, **DuckDB wins**:

- **Identifier quoting and case-sensitivity.** DuckDB matches identifiers
  case-insensitively whether or not they are quoted; Snowflake folds unquoted
  names to upper case and treats quoted ones as case-sensitive. We follow DuckDB
  — see `ident::ident_matches`, and TECH-DEBT #25/#28 for the sites migrated onto
  it.
- **Name resolution.** Schema/search-path semantics follow DuckDB's rules, not
  Snowflake's current-schema-only rule.
- **Anything else that is a property of the dialect** rather than of semantic
  views — literal syntax, type names and coercion, `NULL` ordering, collation.

So "Snowflake does X" settles a question about semantic-view behaviour, and does
**not** settle a question about identifiers, resolution or dialect. When a
Snowflake behaviour can only be reproduced by breaking a DuckDB convention, keep
the convention and record the divergence in TECH-DEBT rather than importing the
Snowflake rule wholesale.

## Quality Gate

**All phases must pass the full test suite before verification can be marked complete.**

The verification command is:

```bash
just test-all
```

This runs: Rust unit tests, property-based tests, SQL logic tests (sqllogictest), and DuckLake CI tests.

Individual test commands:
- `cargo test` — Rust unit + proptest + doc tests
- `just test-sql` — SQL logic tests via sqllogictest runner (requires `just build` first)
- `just test-ducklake-ci` — DuckLake integration tests

A phase verification that only runs `cargo test` is **incomplete** — sqllogictest covers integration paths that Rust tests do not (e.g., type dispatch through the full extension load → DDL → query pipeline).

**Before pushing to main**, run the full CI mirror:

```bash
just ci
```

This adds linting (clippy pedantic + fmt + cargo-deny) and fuzz target compilation checks on top of `test-all`. The Rust toolchain version is pinned in `rust-toolchain.toml` and bumped automatically via Dependabot.

## Bug Fixing & Refactoring Discipline

**Fix bugs test-first.** Before changing any production code to fix a bug, first
write a test that reproduces the issue and **fails** against the current code
(run it, confirm the red). Only then write the fix, and confirm the same test
goes green. This proves the bug was real, that the fix actually addresses it, and
leaves a permanent regression guard behind. Prefer the layer that most directly
exercises the defect — a `#[cfg(test)]`/`tests_*.rs` unit test for logic, a
sqllogictest for anything on the extension-load → DDL → query path (added to
`test/sql/TEST_LIST`). A "fix" landed without a failing-first test is incomplete.

**Confirm the red for *each* case, not just the run.** The sqllogictest runner stops a file at
its first failing statement, so N new cases in one `.test` file yield exactly one observed red —
the other N-1 are unproven, and any that would have passed anyway (a vacuous test) look
identical. The same halt masks regressions later: a break in the second case stays hidden until
the first is repaired. When one fix needs several cases, either give each its own unit test so
they report independently, or verify the set by reverting the fix and checking every case goes
red. Claiming "confirmed red" for cases you did not individually watch fail is the same false
green as the exit-code trap under "Build/test command rules" below.

**Refactor coverage-forward.** A refactor must never silently reduce test
coverage. Be alert for changes that quietly remove coverage — deleting or
weakening an assertion, gating a test behind a feature/flag that CI doesn't
exercise, dropping a `.test` file from `TEST_LIST`, narrowing a proptest
generator so edge cases no longer occur, or asserting equality on a field the
generator never populates (a vacuous check). When you move or rewrite code,
carry its tests with it and strive to strengthen them; if a change genuinely
must drop a test, say so explicitly and explain why rather than letting it vanish
in a diff.

**A new query-semantics feature must reach the numeric oracles, not just fixed
examples.** Anything that changes what number a query returns — a new query
parameter, a new aggregation shape, a new join or CTE topology — is not
adequately covered by hand-written `.test` rows alone. Hand-picked examples
check the cases you thought of; the differential proptests in `tests/`
(`differential_proptest`, `star_schema_proptest`, `multi_hop_join_proptest`,
`semi_additive_proptest`, `window_metric_proptest`) check the cases you didn't,
against an independently-formulated oracle. Extend at least one of them in the
same change: generate the new construct and mirror it into that harness's oracle

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anentropic/duckdb-semantic-views](https://github.com/anentropic/duckdb-semantic-views) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
