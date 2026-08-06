---
trigger: always_on
description: handles statement events, streamed INSERT rows, PostgreSQL COPY data, and
---

# Repository guidance

`sql-splitter` is a Rust CLI and library for inspecting and transforming SQL
dumps. It supports MySQL/MariaDB, PostgreSQL, SQLite, and MSSQL workflows,
including splitting, analysis, conversion, validation, sampling, sharding,
redaction, synthetic-data generation, schema graphs, ordering, diffing, merging,
and optional DuckDB queries.

This file is an agent-oriented map of the repository. Keep volatile CLI details,
benchmarks, and release steps in their authoritative locations instead of
duplicating them here.

## Sources of truth

Use these sources in descending order of authority:

1. Rust source and tests for behavior.
2. `sql-splitter <command> --help` for the current CLI.
3. `just --list` and `justfile` for development commands.
4. `website/src/content/docs/` for maintained user and contributor docs.
5. `README.md` and `skills/sql-splitter/SKILL.md` for secondary user- and
   agent-facing summaries.

When behavior changes, update every affected user-facing source in the same
change. Do not preserve stale prose merely because it appears in this file.

## Before editing

- Run `git status --short` and preserve unrelated worktree changes.
- Read the implementation and nearby tests before changing behavior.
- Search for the same option, command, or concept across `README.md`,
  `website/src/content/docs/`, and `skills/sql-splitter/SKILL.md`.
- Use `cargo run -- <command> --help` when validating CLI examples or flags.

## Development commands

Run `just` or `just --list` for the complete, current command list.

| Task                               | Command                 |
| ---------------------------------- | ----------------------- |
| Debug build                        | `just build`            |
| Release build                      | `just release`          |
| Type-check                         | `just check`            |
| Rust and Markdown formatting       | `just fmt`              |
| Clippy with warnings denied        | `just clippy`           |
| Full nextest suite                 | `just test`             |
| Criterion benchmarks               | `just bench`            |
| Real-world ignored tests           | `just verify-realworld` |
| Generate and validate JSON schemas | `just schemas`          |
| Generate man pages                 | `just man`              |
| Website checks                     | `just website-lint`     |
| Release preparation                | `just release-prepare`  |

Useful direct commands:

```bash
cargo run -- <command> --help
cargo nextest run <filter>
cargo test --doc
cargo fmt --all -- --check
cargo clippy -- -D warnings
```

`just fmt` rewrites Rust and all Markdown files. Use the non-mutating formatting
checks when you only need verification or when the worktree contains unrelated
Markdown changes.

## Verification expectations

Match verification to the change:

- Rust behavior: run focused tests first, then `just test` when practical.
- Parser or dialect behavior: run parser tests plus the relevant integration
  and regression tests.
- CLI arguments or help: run the command's help, then run
  `cargo nextest run --test cli_help_test`.
- Public library examples: run `cargo test --doc` in addition to nextest.
- Lint-sensitive Rust changes: run `cargo fmt --all -- --check` and
  `just clippy`.
- JSON output or synthetic config schema changes: run `just schemas` and check
  in all generated copies.
- Website changes: run `just website-lint`; use `just website-build` when routes,
  components, configuration, or generated assets change.
- Performance-sensitive changes: run the focused Criterion benchmark or the
  relevant script under `scripts/`; record the environment with any numbers.

CI behavior is defined in `.github/workflows/`. Do not infer current CI coverage
from old benchmark or test-count snapshots.

## Architecture map

### Entry points and command layer

- `src/main.rs` is the binary entry point.
- `src/lib.rs` exposes the library API. The binary currently declares the same
  source modules separately, so check both crate targets when changing module
  visibility or conditional compilation.
- `src/cmd/` owns Clap arguments, input validation, command orchestration, and
  process exit codes. Business logic belongs in the domain modules, not in the
  argument structs.
- `src/cmd/common.rs` and `src/cmd/glob_util.rs` contain shared command plumbing.

The command enum in `src/cmd/mod.rs` is the authoritative list of user-facing
subcommands. Some developer commands, such as schema and man-page generation,
are intentionally hidden from normal help.

### SQL input, parsing, and output

- `src/parser/` contains dialect detection and the streaming SQL parser. It
  handles statement events, streamed INSERT rows, PostgreSQL COPY data, and
  MSSQL-specific syntax.
- `src/splitter/` coordinates input decoding, parser events, table filtering,
  archive/compression behavior, and the writer pipeline.
- `src/writer/` owns parallel per-table output, buffering, I/O profiles, and the
  adaptive controller.
- `src/archive.rs` and `src/zip_input.rs` handle feature-gated archive input and
  output.
- `src/copy_data.rs` and `src/parser/{mysql_insert,postgres_copy}.rs` contain
  row-level parsing used by several commands.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HelgeSverre/sql-splitter](https://github.com/HelgeSverre/sql-splitter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
