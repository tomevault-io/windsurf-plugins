---
trigger: always_on
description: This repository contains both Ruff (a Python linter and formatter) and ty (a Python type checker). The crates follow a naming convention: `ruff_*` for Ruff-specific code and `ty_*` for ty-specific code. ty reuses several Ruff crates, including the Python parser (`ruff_python_parser`) and AST definitions (`ruff_python_ast`).
---

# Ruff Repository

This repository contains both Ruff (a Python linter and formatter) and ty (a Python type checker). The crates follow a naming convention: `ruff_*` for Ruff-specific code and `ty_*` for ty-specific code. ty reuses several Ruff crates, including the Python parser (`ruff_python_parser`) and AST definitions (`ruff_python_ast`).

## Code reviews

When reviewing a branch or pull request, be deliberately nitpicky. Report not
only bugs and regressions, but also architectural and maintenance risks, weak
test coverage, unclear code, unnecessary complexity, and meaningful style or
consistency issues. Order findings by severity, cite files and lines, and
distinguish blockers from non-blocking improvements. Number each review point
for easy reference in subsequent review discussion.

## Running Tests

Run all tests (using `nextest` for faster execution, setting `CARGO_PROFILE_DEV_OPT_LEVEL=1 CARGO_PROFILE_DEV_DEBUG="line-tables-only"` to enable optimizations while retaining some debug info, and setting `INSTA_FORCE_PASS=1 INSTA_UPDATE=always MDTEST_UPDATE_SNAPSHOTS=1` to ensure all snapshots are updated):

```sh
CARGO_PROFILE_DEV_OPT_LEVEL=1 INSTA_FORCE_PASS=1 INSTA_UPDATE=always CARGO_PROFILE_DEV_DEBUG="line-tables-only" MDTEST_UPDATE_SNAPSHOTS=1 cargo nextest run
```

Run tests for a specific crate:

```sh
CARGO_PROFILE_DEV_OPT_LEVEL=1 INSTA_FORCE_PASS=1 INSTA_UPDATE=always CARGO_PROFILE_DEV_DEBUG="line-tables-only" MDTEST_UPDATE_SNAPSHOTS=1 cargo nextest run -p ty_python_semantic
```

Run a single mdtest file. The path to the mdtest file should be relative to the `crates/ty_python_semantic/resources/mdtest` folder:

```sh
CARGO_PROFILE_DEV_OPT_LEVEL=1 INSTA_FORCE_PASS=1 INSTA_UPDATE=always CARGO_PROFILE_DEV_DEBUG="line-tables-only" MDTEST_UPDATE_SNAPSHOTS=1 cargo nextest run -p ty_python_semantic -- mdtest::<path/to/mdtest_file.md>
```

To run a specific mdtest within a file, use a substring of the Markdown header text as `MDTEST_TEST_FILTER`. Only use this if it's necessary to isolate a single test case:

```sh
MDTEST_TEST_FILTER="<filter>" CARGO_PROFILE_DEV_OPT_LEVEL=1 INSTA_FORCE_PASS=1 INSTA_UPDATE=always CARGO_PROFILE_DEV_DEBUG="line-tables-only" MDTEST_UPDATE_SNAPSHOTS=1 cargo nextest run -p ty_python_semantic -- mdtest::<path/to/mdtest_file.md>
```

### Fallback without nextest

If `cargo nextest` is not available, use `cargo test` with the same environment variables:

```sh
# Run all tests.
CARGO_PROFILE_DEV_OPT_LEVEL=1 INSTA_FORCE_PASS=1 INSTA_UPDATE=always CARGO_PROFILE_DEV_DEBUG="line-tables-only" MDTEST_UPDATE_SNAPSHOTS=1 cargo test

# Run tests for a specific crate.
CARGO_PROFILE_DEV_OPT_LEVEL=1 INSTA_FORCE_PASS=1 INSTA_UPDATE=always CARGO_PROFILE_DEV_DEBUG="line-tables-only" MDTEST_UPDATE_SNAPSHOTS=1 cargo test -p ty_python_semantic

# Run a single mdtest file.
CARGO_PROFILE_DEV_OPT_LEVEL=1 INSTA_FORCE_PASS=1 INSTA_UPDATE=always CARGO_PROFILE_DEV_DEBUG="line-tables-only" MDTEST_UPDATE_SNAPSHOTS=1 cargo test -p ty_python_semantic --test mdtest -- <path/to/mdtest_file.md>

# Run a specific mdtest within a file.
MDTEST_TEST_FILTER="<filter>" CARGO_PROFILE_DEV_OPT_LEVEL=1 INSTA_FORCE_PASS=1 INSTA_UPDATE=always CARGO_PROFILE_DEV_DEBUG="line-tables-only" MDTEST_UPDATE_SNAPSHOTS=1 cargo test -p ty_python_semantic --test mdtest -- <path/to/mdtest_file.md>
```

### Snapshot updates

After running the tests, always review the contents of any snapshots that have been added or updated.

When running tests with `INSTA_FORCE_PASS=1`, check for `.pending-snap` files if any affected tests use inline snapshots.

Never edit snapshot files or inline snapshot bodies manually. Regenerate them by running the relevant tests with the snapshot-update environment variables documented above, then review the generated diff.

## Running Clippy

```sh
cargo clippy --workspace --all-targets --all-features -- -D warnings
```

## Running Debug Builds

Use debug builds (not `--release`) when developing, as release builds lack debug assertions and have slower compile times.

Run Ruff:

```sh
cargo run --bin ruff -- check path/to/file.py
```

Run ty:

```sh
cargo run --bin ty -- check path/to/file.py
```

## Working on ty

The guidance in this section applies to edits to `ty*` crates, reviews of ty PRs, or other work when the ty type checker has been specifically mentioned by the user.

### Related skills

When the task matches a more specific ty workflow, also read and follow that skill from the repository root:

- Diagnostic changes, diagnostic message changes, or diagnostic reviews: `.agents/skills/adding-ty-diagnostics/SKILL.md`.
- Ecosystem report summaries: `.agents/skills/summarise-ecosystem-results/SKILL.md`.
- Reproducing, investigating, or minimizing ecosystem or primer differences: `.agents/skills/minimizing-ty-ecosystem-changes/SKILL.md`.

### Ad hoc reproductions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [astral-sh/ruff](https://github.com/astral-sh/ruff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
