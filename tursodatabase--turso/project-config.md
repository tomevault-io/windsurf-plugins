---
trigger: always_on
description: SQLite rewrite in Rust. 40+ crate workspace.
---

# Turso Agent Guidelines

SQLite rewrite in Rust. 40+ crate workspace.

## Quick Reference

```bash
cargo build                    # build. never build with --release
cargo test                     # rust unit/integration tests
cargo fmt                      # format (required)
cargo clippy --workspace --all-features --all-targets -- --deny=warnings  # lint
cargo run -q --bin tursodb -- -q # run the interactive cli. never run with --release

make test                      # TCL compat + sqlite3 + extensions + MVCC
make test-single TEST=foo.test # single TCL test
make -C sqlite/conformance run-rust ARGS='--snapshot-filter __never__'  # sqltest runner (preferred for new tests)
CI=1 make -C sqlite/conformance run-rust  # use only if snapshot tests are required

scripts/diff.sh "SQL" [label]  # compare sqlite3 vs tursodb output
```

## Testing

### Running Tests

- `cargo test` - Rust unit and integration tests
- `make test` - broad compatibility suite (TCL, sqlite3, extensions, MVCC)
- `make test-single TEST=foo.test` - single legacy TCL test
- `make -C sqlite/conformance run-rust ARGS='--snapshot-filter __never__'` - preferred `.sqltest` runner for new coverage
- `CI=1 make -C sqlite/conformance run-rust` - only when snapshot tests are required

### Test Organization

Default: add coverage to the narrowest existing test harness that can express the bug. Prefer extending an existing test file or directory over creating a new one.

- `sqlite/conformance/sqlite-sqltests/` - preferred for SQL conformance coverage. These tests run the same scenario against both Turso and SQLite, so use them first for parser, planner, executor, and SQL semantics work that fits the `.sqltest` DSL.
- `tests/integration/` - primary fallback when the behavior cannot be expressed cleanly in `.sqltest`. Put API-level regressions, multi-connection orchestration, storage assertions, injected failures, timeout behavior, and other Rust-driven scenarios here.
- `sqlite/conformance/upstream/` - imported upstream SQLite golden tests. Do not modify these for Turso behavior changes; use them as fixed compatibility coverage, and only touch them for intentional upstream sync or harness maintenance.
- `testing/cli_tests/` - CLI-focused Python coverage for shell behavior and end-to-end command workflows.
- `tests/fuzz/` - minimized fuzz regressions and targeted edge cases that are easier to keep as Rust tests.
- `testing/simulator/` and `testing/concurrent-simulator/` - deterministic concurrency, scheduling, and failure-injection coverage for state-machine and I/O correctness.
- `testing/differential-oracle/` and `testing/stress/` - differential and long-running stress tooling. Use these for deeper investigation or specialized validation, not as the first stop for a focused regression test.

## Structure

```
limbo/
├── core/           # Database engine (translate/, storage/, vdbe/, io/, mvcc/)
├── sqlite/
│   └── parser/     # SQL parser (lexer, AST, grammar)
├── cli/            # tursodb CLI (REPL, MCP server, sync server)
├── bindings/       # Python, JS, Java, .NET, Go, Rust
├── extensions/     # crypto, regexp, csv, fuzzy, ipaddr, percentile
├── testing/        # simulator/, concurrent-simulator/, differential-oracle/
├── sync/           # engine/, sdk-kit/ (Turso Cloud sync)
├── sdk-kit/        # High-level SDK abstraction
└── tools/          # dbhash utility
```

## Where to Look

| Task | Location | Notes |
|------|----------|-------|
| Query execution | `core/vdbe/execute.rs` | 12k LOC bytecode interpreter |
| SQL compilation | `core/translate/` | AST → bytecode, optimizer in `optimizer/` |
| B-tree/pages | `core/storage/btree.rs` | 10k LOC, SQLite-compatible format |
| WAL/durability | `core/storage/wal.rs` | Write-ahead log, checkpointing |
| SQL parsing | `sqlite/parser/src/parser.rs` | 11k LOC recursive descent |
| Add extension | `extensions/core/` | ExtensionApi, scalar/aggregate/vtab traits |
| Add binding | `bindings/` | PyO3, NAPI, JNI, FRB, CGO patterns |
| Deterministic tests | `testing/simulator/` | Fault injection, differential testing |
| New SQL tests | `sqlite/conformance/sqlite-sqltests/` | `.sqltest` format preferred |
| Quick sqlite3 diff | `scripts/diff.sh` | Compare sqlite3 vs tursodb output for a query |
| MVCC testing REPL | `cli/mvcc_repl.rs` | Multi-conn concurrent txn testing REPL        |

## Guides

- **[Testing](docs/agent-guides/testing.md)** - test types, when to use, how to write
- **[Code Quality](docs/agent-guides/code-quality.md)** - correctness rules, Rust patterns, comments
- **[Debugging](docs/agent-guides/debugging.md)** - bytecode comparison, logging, sanitizers
- **[PR Workflow](docs/agent-guides/pr-workflow.md)** - commits, CI, dependencies
- **[Transaction Correctness](docs/agent-guides/transaction-correctness.md)** - WAL, checkpointing, concurrency
- **[Storage Format](docs/agent-guides/storage-format.md)** - file format, B-trees, pages
- **[Async I/O Model](docs/agent-guides/async-io-model.md)** - IOResult, state machines, re-entrancy
- **[MVCC](docs/agent-guides/mvcc.md)** - experimental multi-version concurrency (WIP)

## Commit Messages

Use an optional component scope followed by a lowercase imperative summary with
no trailing period:

```text
[scope: ]<imperative summary>


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tursodatabase/turso](https://github.com/tursodatabase/turso) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
