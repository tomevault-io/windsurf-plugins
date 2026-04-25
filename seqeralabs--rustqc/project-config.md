---
trigger: always_on
description: > Fast quality control tools for sequencing data, written in Rust. The
---

# AGENTS.md — RustQC

> Fast quality control tools for sequencing data, written in Rust. The
> **`rustqc rna`** command runs all RNA-Seq QC analyses in a single pass:
>
> - **dupRadar** duplicate rate analysis (reimplementation of
>   [dupRadar](https://github.com/ssayols/dupRadar))
> - **featureCounts**-compatible gene-level read counting with biotype summaries
> - **8 RSeQC tools** integrated as built-in analyses: bam_stat, infer_experiment,
>   read_duplication, read_distribution, junction_annotation, junction_saturation,
>   inner_distance, and TIN (reimplementations of [RSeQC](https://rseqc.sourceforge.net/))
> - **preseq** library complexity extrapolation (reimplementation of
>   [preseq](https://github.com/smithlabcode/preseq) `lc_extrap`)
> - **samtools-compatible** flagstat, idxstats, and full stats output
> - **Gene body coverage** profiling with Qualimap-compatible output
>
> Binary crate (`rustqc`), Rust edition 2021.

## Build / Lint / Test Commands

```bash
# Build
cargo build              # debug build
cargo build --release    # optimized release build (LTO, strip, opt-level 3)

# Format (enforced in CI — default rustfmt, no config file)
cargo fmt                # auto-format
cargo fmt --check        # check only (CI uses this)

# Lint (enforced in CI — default clippy, warnings are errors)
cargo clippy -- -D warnings

# Pre-commit hooks (runs fmt --check + clippy + file hygiene)
# Install: brew install prek (or cargo install prek)
prek install          # set up git hooks
prek run --all-files  # run all hooks manually

# Test — all unit + integration tests
cargo test               # debug mode
cargo test --release     # release mode (CI uses this)

# Run a single test by name (substring match)
cargo test test_dup_rate_calculation
cargo test test_dup_matrix_exact_match -- --nocapture   # with stdout

# Run only unit tests (skip integration tests)
cargo test --lib

# Run only integration tests
cargo test --test integration_test

# Run a specific integration test
cargo test --test integration_test test_intercept_slope_match
```

## Project Structure

```
src/
  main.rs           — Entry point, dispatches subcommands
  cli.rs            — CLI argument parsing (clap derive, subcommand structure)
  config.rs         — YAML configuration loading (serde), nested tool configs
  io.rs             — Shared I/O utilities (gzip-transparent file reading)
  gtf.rs            — GTF annotation file parser (with configurable attribute extraction)
  rna/
    mod.rs          — Re-exports all submodules (dupradar, featurecounts, rseqc, bam_flags, cpp_rng, preseq, qualimap)
    bam_flags.rs    — BAM flag constants
    cpp_rng.rs      — C++ RNG FFI shim for preseq bootstrap reproducibility
    dupradar/
      mod.rs        — Re-exports counting, dupmatrix, fitting, plots
      counting.rs   — BAM read counting engine (largest module)
      dupmatrix.rs  — Duplication matrix construction & TSV output
      fitting.rs    — Logistic regression via IRLS
      plots.rs      — Plot generation: density scatter, boxplot, histogram
    featurecounts/
      mod.rs        — Re-exports output
      output.rs     — featureCounts-format output & biotype counting
    preseq.rs       — preseq lc_extrap library complexity extrapolation
    qualimap/
      mod.rs        — Re-exports all Qualimap modules
      accumulator.rs — Gene body coverage accumulation logic
      coverage.rs   — Per-base coverage computation
      index.rs      — Transcript index for coverage profiling
      output.rs     — Qualimap-compatible output file generation
      plots.rs      — Gene body coverage plot generation (PNG/SVG)
      report.rs     — Qualimap HTML report generation
    rseqc/
      mod.rs                — Re-exports all RSeQC modules + common helpers
      accumulators.rs       — Shared RSeQC accumulator infrastructure (read dispatch)
      common.rs             — Shared junction/intron extraction, from_genes builders
      bam_stat.rs           — bam_stat.py reimplementation
      flagstat.rs           — samtools flagstat-compatible output
      idxstats.rs           — samtools idxstats-compatible output
      infer_experiment.rs   — infer_experiment.py reimplementation
      inner_distance.rs     — inner_distance.py reimplementation
      junction_annotation.rs — junction_annotation.py reimplementation
      junction_saturation.rs — junction_saturation.py reimplementation
      plots.rs              — RSeQC plot generation (duplication, junctions, etc.)
      read_distribution.rs  — read_distribution.py reimplementation
      read_duplication.rs   — read_duplication.py reimplementation
      stats.rs              — samtools stats full output (SN + all histogram sections)
      tin.rs                — TIN (Transcript Integrity Number) analysis
tests/
  integration_test.rs  — 12 integration tests vs R dupRadar reference output
  data/                — Test BAM/GTF input files
  expected/            — R-generated reference outputs
  create_test_data.R   — R script to regenerate test data + references
```

Nested module structure — top-level modules (`cli`, `config`, `io`, `gtf`, `rna`) declared
in `main.rs`, no `lib.rs`. The `rna` module contains sub-modules for each tool group.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [seqeralabs/RustQC](https://github.com/seqeralabs/RustQC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
