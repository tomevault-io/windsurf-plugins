---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`chelae` is a FASTQ trimming and filtering toolkit written in Rust. The name is the plural of *chela*, the pincer-like claws of crustaceans — apt for a tool that clips reads.

Currently the crate ships a single subcommand, `chelae trim`, which runs the common short-read preprocessing pipeline (poly-G trim → adapter trim → read-structure hard-trim + UMI extraction → poly-X trim → sliding-window quality trim → length/N-base/mean-quality/low-qual-fraction filters) on SE or PE FASTQ data. Outputs are BGZF-compressed FASTQ plus a fastp-compatible JSON report for MultiQC consumption.

Read-structure runs after adapter trim so tail-skip segments (like the `10S` in `+T10S`) drop bases from the post-adapter template, not from the raw read where the adapter step would usually have removed them already. If a read is shorter than the read-structure's fixed-length segments require (either originally or after adapter trim), the pair is dropped and counted under `reads_filtered_length`.

The crate structure (multi-subcommand CLI with `enum_dispatch`) is deliberately kept open-ended so additional trimming/filtering utilities can be added as siblings of `trim` without a CLI shape change.

## Origin — very important

**`chelae` was extracted from [`fqtk`](https://github.com/fulcrumgenomics/fqtk) on 2026-04-21.** The entire `chelae trim` implementation — code, design decisions, performance work, tests — was developed as `fqtk trim` on the `tf_trim` branch in that repo and then split into this standalone crate. If you are working on `chelae` and need context that isn't in this file:

1. The full design history is in the [`fqtk`](https://github.com/fulcrumgenomics/fqtk) repo's git log and GitHub PR history, branch `tf_trim`.
2. Tim keeps a local checkout at `/Users/tfenne/work/open-source/fqtk` (on branch `tf_trim`) — all the incremental commits that became the initial chelae import are visible there (pre-squash, on the `tf_trim_backup` branch locally), along with their commit messages. If you need to understand *why* a design choice was made, that's the source of truth.
3. Auto-memory from fqtk development was pre-seeded into this project's memory directory at the time of extraction. See `~/.claude/projects/-Users-tfenne-work-open-source-chelae/memory/` for feedback rules, project notes, and references.

The `chelae` initial import is a three-commit squash of 30+ fqtk commits. The squashed history omits detail intentionally — when you need "why did we do it this way", look in the fqtk backup branch, not here.

## Build & Test Commands

```bash
# Full verification (format + clippy + tests) — run before pushing.
bash ci/check.sh

# Individual steps
cargo fmt --all
cargo clippy --all-features --all-targets -- -D warnings
cargo test

# Run a single test
cargo test <test_name>

# Build release binary
cargo build --release
```

CI (`.github/workflows/build_and_test.yml`) additionally runs `src/scripts/precommit.sh` (the same checks with `--locked`).

The README has a hand-curated **Options** table that summarizes every `chelae trim` flag. When you add, remove, rename, or materially change a CLI option, update that table in `README.md` to match. The `chelae trim --help` output remains the authoritative reference; the README table is the short-form pointer.

## Toolchain

Pinned to Rust 1.95 via `rust-toolchain.toml`. Format settings: `max_width = 100`, `use_small_heuristics = "max"` (see `rustfmt.toml`).

## Build Targeting

### x86_64: cargo-multivers

x86_64 release binaries are packaged via [`cargo-multivers`](https://github.com/ronnychevalier/cargo-multivers) into a single launcher that embeds three CPU-specific builds and dispatches to the best match at startup. See `[package.metadata.multivers.x86_64]` in `Cargo.toml`:

```toml
cpus = ["x86-64", "x86-64-v2", "x86-64-v4"]
```

- `x86-64` — SSE2 baseline, any 64-bit x86 (2003+)
- `x86-64-v2` — SSE4.2 + POPCNT (2008+). Captures nearly all of the scalar codegen win
- `x86-64-v4` — AVX-512F/BW/CD/DQ/VL (2017+ server / 2022+ consumer). ~1% additional win

We intentionally skip `x86-64-v3`: our Granite Rapids benchmark showed v2 and v3 within measurement noise on chelae's workload. The historical "x86-64-v3 wins 6% over baseline" finding is actually a v1→v2 win; v2→v3 contributes ~0. Including v3 would bloat the binary without buying anything.

Variants are delta-compressed (`gdelta`) + lz4. Total binary is ~3.7 MB (vs ~2.9 MB for a single-variant build). Startup adds ~0.2 s for decompression + `memfd_create + exec` — negligible for chelae's batch workload.

The cargo-multivers runner sorts variants by feature count descending and picks the first match — so v4 runs on capable hardware, falling back to v2 on pre-AVX-512 systems and v1 on pre-SSE4.2 systems.

### Build commands

- x86_64: `cargo multivers --profile dist`
- aarch64: `cargo build --profile dist`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fulcrumgenomics/chelae](https://github.com/fulcrumgenomics/chelae) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
