---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

fgumi is a high-performance Rust CLI tool for UMI (Unique Molecular Identifier) processing in sequencing data. It provides extraction, grouping, correction, and consensus calling functionality.

## Build and Test Commands

```bash
# Build
cargo build --release

# Run all tests (recommended - uses nextest)
cargo ci-test

# Run tests with test-utils feature
cargo t

# Check formatting
cargo ci-fmt

# Run linting
cargo ci-lint

# Run all CI checks
cargo ci-test && cargo ci-fmt && cargo ci-lint

# Run a single test
cargo nextest run test_name

# Run benchmarks
cargo bench
```

## Features

- `compare` - Enable compare subcommand (developer tools)
- `simulate` - Enable simulate command for test data generation
- `profile-adjacency` - Enable profiling output for adjacency UMI assigner

Build with features: `cargo build --release --features compare,simulate`

## Code Architecture

### Entry Points
- `src/main.rs` - CLI entry point using clap with enum_dispatch for command routing
- `src/lib/mod.rs` - Library crate (`fgumi_lib`) with all core functionality

### Library Modules (`src/lib/`)

**Core UMI Processing:**
- `umi/` - UMI assignment strategies: identity, edit-distance, adjacency, paired
- `consensus/` - Consensus callers: simplex, duplex, codec
- `grouper.rs` - Read grouping by UMI

**I/O:**
- `bam_io/` - BAM file reading/writing helpers
- `bgzf_reader/`, `bgzf_writer/` - BGZF compression I/O
- `sam/` - SAM/BAM alignment utilities and tag manipulation

**Utilities:**
- `bitenc.rs` - 2-bit DNA encoding (4 bases per byte, 32 per u64)
- `metrics/` - QC metrics for duplex, consensus, grouping
- `validation.rs` - Input file and parameter validation
- `progress.rs` - Progress tracking
- `reference.rs` - Reference genome/dict file handling

**Specialized:**
- `clipper.rs` - Overlapping read pair clipping
- `template.rs` - Template-based read grouping
- `vendored/bam_codec/` - Isolated vendored BAM compression codec

### Commands (`src/commands/`)

Commands implement the `Command` trait dispatched via enum:

1. **Extraction:** `extract` - Extract UMIs from FASTQ to unmapped BAM
2. **Alignment Support:** `fastq`, `zipper`, `sort`
3. **UMI Operations:** `correct`, `group`, `dedup`
4. **Consensus:** `simplex`, `duplex`, `codec`
5. **Post-consensus:** `filter`, `clip`, `duplex-metrics`, `review`
6. **Utilities:** `downsample`, `compare` (feature-gated), `simulate` (feature-gated)

### Key Design Patterns

- **Enum Dispatch:** Commands use `enum_dispatch` for trait-based routing
- **Streaming I/O:** Large files processed without full loading
- **Thread Pooling:** Work-stealing with per-command thread optimization
- **2-bit Encoding:** DNA bases packed efficiently for fast operations

## Development Practices

### Pre-commit Hooks
Install with `./scripts/install-hooks.sh`. Runs `cargo ci-fmt` and `cargo ci-lint` before commits.

To include tests: `FGUMI_PRECOMMIT_TEST=1 git commit`

### Commit Messages
Use conventional commits format: `<type>[(scope)][!]: <description>`

Types: `feat`, `fix`, `build`, `chore`, `ci`, `config`, `docs`, `example`, `perf`, `refactor`, `style`, `test`

### Branch Naming
`<issue-number>/<user>/<type>-<description>` (e.g., `42/jdidion/fix-fibonacci-calculation`)

### Testing
- Unit tests alongside source in `src/`
- Integration tests in `tests/integration/`
- Uses `rstest` for parameterized tests, `proptest` for property-based testing
- `criterion` for benchmarks in `benches/`

## Rust Version

Minimum supported and pinned build version: 1.93.0 (edition 2024).

fgumi supports exactly the Rust it is developed and tested on: `rust-version`
in `Cargo.toml` (the published MSRV) is kept identical to the `channel` in
`rust-toolchain.toml`, and CI's `msrv-lockstep` job fails if they drift. When
raising the toolchain, bump `rust-version` in the same change and adopt any
idioms the newer compiler unlocks (e.g. let-chains) rather than suppressing the
clippy lints that flag them.

## Allocator

Uses `mimalloc` as global allocator for performance.

## Unsafe Code

`#![deny(unsafe_code)]` is set at the crate root of every workspace member. Targeted
`#[allow(unsafe_code)]` blocks are permitted only at the documented sites listed below;
any new `unsafe` block requires updating this section with a written justification.

### Approved non-stdlib FFI exceptions

The following external FFI calls are approved because they back core infrastructure:

- **`libmimalloc_sys`** (`crates/fgumi-sort/src/memory_probe.rs`) — mimalloc is the configured
  global allocator. The FFI wrappers (`force_mi_collect`, `process_rss_bytes`, and the
  `memory-debug`-gated `print_mi_stats` calling `mi_stats_print_out`) are isolated to
  a single `#[allow(unsafe_code)]` sub-module. mimalloc synchronizes these calls
  internally, so they are safe to invoke concurrently.
- **`mach2`** (`crates/fgumi-sort/src/memory_probe.rs`, macOS only) — `task_info(TASK_VM_INFO)`
  is the only way to read `phys_footprint` (the RSS metric mimalloc reports accurately).
  Isolated to the same sub-module as the mimalloc FFI.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fulcrumgenomics/fgumi](https://github.com/fulcrumgenomics/fgumi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
