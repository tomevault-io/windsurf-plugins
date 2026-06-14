---
trigger: always_on
description: description: "Functionally complete Git reimplmentation in idiomatic, library focused Rust"
---


---
description: "Functionally complete Git reimplmentation in idiomatic, library focused Rust"
alwaysApply: true
---

# AGENTS.md — Working on Grit

A complete rewrite of Git in idiomatic, library-focused Rust code. This file is the durable build contract for autonomous runs.

## Product Intent

Grit is a from-scratch reimplementation of Git in idiomatic, library-oriented Rust.
The goal: pass the entire upstream Git test suite.

## Quick Start

```bash
# Build
cargo build --release -p grit-cli

# Run a single test file
./scripts/run-tests.sh t3200-branch.sh

# Run one group (e.g. t1xxx)
./scripts/run-tests.sh t1

# Full harness (in-scope files only)
./scripts/run-tests.sh
```

## Testing pipeline (harness)

Upstream-style tests live in `tests/` and are driven by **`scripts/run-tests.sh`**. Per-file status and last-run counts live in per-test TOML files at **`data/tests/<group>/<stem>.toml`** (e.g. `data/tests/t0/t0000-basic.toml`). Dashboards **`docs/index.html`** (homepage progress section), **`docs/progress/index.html`**, **`docs/testfiles.html`**, and **`docs/test-progress.svg`** (README progress badge) are generated from that tree — only when you pass `--dashboard` to `run-tests.sh` or run `scripts/generate-dashboard-from-test-files.py` directly.

**Flow:**

1. **`scripts/run-tests.sh`** — Runs the requested files (single `.sh`, group prefix like `t1`, or all rows with `in_scope=yes`). Rows with **`in_scope=skip`** are never run.
2. **`scripts/generate-dashboard-from-test-files.py`** — Regenerates **`docs/index.html`** progress metrics, **`docs/progress/index.html`**, **`docs/testfiles.html`**, and **`docs/test-progress.svg`**.

To skip a file manually, set **`in_scope = "skip"`** in that test's TOML (`data/tests/<group>/<stem>.toml`). Skipped files are omitted from runs and from aggregate counts on the main dashboard.

Full detail: **TESTING.md**.

## The One Rule

**Fix grit Rust code to make upstream tests pass. Do not modify tests.**

The only exception: flipping `test_expect_failure` → `test_expect_success` when
you've fixed the underlying bug.

## Source of Truth

The canonical Git source code we're targeting to replicate the functionality of is in the `git/` subdirectory.

The tests we're trying to make pass with our new implementation is in the `git/t/` directory.

Manpage documentation is located in `git/Documentation` directory as `*.doc` files.

## Licensing Hard Rule — No Copied Expression in `grit-lib`

`grit-lib` is **MIT-licensed**; Git's C source under `git/` is **GPLv2**. To keep
the library clean, this rule is absolute:

**Never copy protected expression from Git's C source into `grit-lib`. Use the
C source only for ideas, methods, interfaces, and behavior.**

- **Allowed** (not protectable): the *algorithm or method* (e.g. Myers diff, the
  approxidate parser, name-hash math), the *interface/behavior* it must produce,
  byte-for-byte *output compatibility*, and *facts* (keyword lists, opcode
  tables, format constants). Reimplement these in your own idiomatic Rust.
- **Forbidden** (protected expression copied verbatim or near-verbatim): Git's
  prose **comments**, multi-line **user-facing message strings**, and code whose
  **structure, naming, and layout** track the C beyond what the method requires.

If Git-identical user-facing text or other copied expression is genuinely
needed, it lives in the **`grit` CLI crate (`grit/src`), which is GPL-2.0** and
may reuse Git's strings and expression — not in `grit-lib`. Have the library
return a **structured/typed error or value**, and render the Git-compatible text
at the CLI boundary.

When porting from `git/`: read the C to understand *what* and *why*, then write
the Rust from that understanding — do not transcribe.

## How to Work

Read **TESTING.md** for the full strategy. The short version:

1. Pick **one test file** that isn't fully passing
2. Run it, study the failures
3. Fix the Rust code in `grit-lib/src/` by default; use `grit/src/` only for CLI parsing, process setup, user-facing output, or thin command wiring.
4. Rebuild (`cargo build --release -p grit-cli`)
5. Re-run until fully passing
6. Refresh results: `./scripts/run-tests.sh <file>.sh` (updates that test's `data/tests/` TOML; add `--dashboard` to regenerate docs)
7. Commit with GitButler (see **Committing** below) with a message like `fix: make t1234-foo fully pass`

### Priority Order

Plumbing first (t0-t1), then core commands (t2-t3), diff (t4), transport (t5),
rev machinery (t6), porcelain (t7), external helpers (t9) last.

Within each category: files closest to fully passing first (quick wins).

### Before Committing Rust Code

```bash
cargo fmt
cargo check # fix warnings
cargo clippy --fix --allow-dirty   # ensure no warnings remain
cargo test -p grit-lib --lib       # unit tests must pass
```

## Looping Rules

There may be several agents working in this directory to coordinate implementation. Work is tracked in **TicGit** (`ti`): run `ti agent` for the full usage guide. Every harness test file that is not yet fully passing has an open ticket tagged `test` plus its family tag (`t0`–`t9`); each ticket lists the failing subtests and how to reproduce.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gitbutlerapp/grit](https://github.com/gitbutlerapp/grit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-14 -->
