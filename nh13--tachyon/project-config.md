---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`tachyon` is a single-binary, zero-dependency Rust probe that measures a host's **memory access rate under whatever contention exists right now**. It exists so benchmark wall times from shared cloud hosts can be told apart from real code regressions: the same unchanged binary measured 18.89 s and 25.01 s on different `m7i.4xlarge` instances, and the score is the control variable that flags such a host.

Read `README.md` (the argument for the design) and `src/lib.rs`'s module doc (the same argument, with the measurements) before changing anything under `src/`.

`CONTRIBUTING.md` is the authority on the change rules, testing rationale, and dependency policy. This file covers what an agent needs on top of that: where things live, and which claims about this codebase are easy to get wrong.

## Commands

```bash
cargo ci-fmt     # fmt --all -- --check
cargo ci-lint    # clippy --all-features --all-targets -- -D warnings
cargo ci-test    # test --locked --all-features
```

These three aliases live in `.cargo/config.toml` and are exactly what CI runs. `--locked` is deliberate — it fails on a stale lockfile rather than silently re-resolving.

Single test: `cargo test chain_is_a_single_full_length_cycle`. Integration tests only: `cargo test --test cli`. Whole crate builds in ~1 s; there is no nextest and no watch setup, on purpose.

Optional pre-commit hook (runs `ci-fmt` + `ci-lint`): `./scripts/install-hooks.sh`.

Manual smoke of the release binary — the thing `cargo test` cannot stand in for, see below:

```bash
cargo build --release --locked
./target/release/tachyon --seconds 2 --working-set-mb 32 --json
```

## The rule that governs changes

**A change to what determines a reading is a change to what past readings mean.** Scores get recorded beside benchmark timings and compared months later; if the access pattern, the chain construction, the default working set, or the units change, every previously recorded score silently becomes incomparable. There is no error message for that.

That covers `src/`, but not only `src/` — `[profile.release]` in `Cargo.toml` is part of the measurement (the optimised build is the one people run), and so is `rust-toolchain.toml`, since a different compiler can generate a different chase.

Classify any such change and say so in the PR description, per `CONTRIBUTING.md`:

- **No change to what is measured** (refactor, docs, CI, tests), or
- **Changes what is measured** — then include before/after scores from the *same machine* and a note on interpreting old readings.

`CHANGELOG.md` carries a `### Measurement` heading for the second kind.

## Architecture

Three files, and the split is meaningful:

- **`src/lib.rs`** — the measurement, and nothing else. `ProbeConfig` in, `ProbeResult` out via `run()`; `slots_for` is also public, for callers sizing a working set. `ProbeResult` stores raw counts (`accesses`, `elapsed`, `threads`, `working_set_bytes`) and derives `million_accesses_per_sec()` (the score) and `ns_per_access()` (the sanity check) on demand, so the stored record cannot disagree with the reported one. Note `working_set_bytes` is the memory actually walked — the request rounded down to whole cache lines — not the value passed in.
- **`src/main.rs`** — hand-rolled arg parsing, JSON and human formatting, exit codes. No measurement logic. `parse_args` returns `Result<Args, String>` so every rejection is testable in-process.
- **`tests/cli.rs`** — runs the built binary via `CARGO_BIN_EXE_tachyon`.

Design invariants, and whether anything actually pins each one:

- **The chain is a single full-length cycle** (Sattolo's algorithm — `j` drawn from `[0, i)`, not `[0, i]`). An arbitrary permutation decomposes into short cycles, and a walker caught in one revisits a handful of lines that then sit in cache — turning a memory probe into a cache probe. *Pinned by `chain_is_a_single_full_length_cycle`.*
- **The chain is a dense `u32` array covering the whole working set** — sixteen slots per 64-byte line, so the allocation is exactly the size requested. Hops are deliberately *not* spread one-per-line; the full-length cycle means a line's other slots come back millions of hops later, long after eviction. *Pinned by `slots_cover_the_whole_working_set`.*
- **Chains are built before the clock starts**, so allocation and permutation are outside the measurement. *Not pinned by any test — structural.*
- **`black_box` guards the chase** against the optimiser deleting pure computation with an unused result. *Not pinned by any test; see below for why that is hard.*
- **The probe must not resemble the code under test.** Sharing code would let a real regression move the probe, cancelling itself out when you normalise. This is why the probe is a pointer chase and not anything drawn from the workloads it is used against. *Not pinnable by a test — a review question.*

## The optimiser-elision trap

Two intuitive things about this failure mode are false, and both were stated wrongly across this repo's docs before being corrected. Do not reintroduce them:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nh13/tachyon](https://github.com/nh13/tachyon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-08 -->
