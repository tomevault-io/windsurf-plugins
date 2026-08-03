---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```sh
# Build
cargo build
cargo build --release

# Run all tests
cargo test

# Run a single test (tests are nested: mod tests > mod <fn-under-test>)
cargo test get_inode_ratio::returns_zero_on_shutdown
cargo test parallel_search::skip_path_skips_only_listed_dirs

# Lint (lint levels live in Cargo.toml [lints]; the flag is belt-and-suspenders)
cargo clippy -- -D warnings

# Format (rustfmt.toml enforces max_width=79)
cargo fmt

# Check formatting without modifying
cargo fmt -- --check

# Regenerate the man page after editing man/findlargedir.1.scd
scdoc < man/findlargedir.1.scd > man/findlargedir.1

# Benchmark findlargedir vs GNU find (Criterion, harness = false)
# Heavy: shallow-clones the Linux kernel into benches/linux_root on first run.
# Set BENCH_WALK_DIR to reuse a checkout; shorten a run with --measurement-time.
# Two groups: walk_linux_kernel (warm) and walk_linux_kernel_cold (drops caches
# via /proc/sys/vm/drop_caches each run — needs root, else skipped with a warning).
cargo bench --bench walk
cargo bench --bench walk -- --measurement-time 20
cargo bench --bench walk -- walk_linux_kernel_cold   # cold-cache group only (root)
```

`edition = "2024"`, `rust-version = "1.88.0"` (MSRV). There is **no** `rust-toolchain.toml` — the toolchain is not pinned, so pin it manually if building elsewhere. Lint levels are centralized in `Cargo.toml`'s `[lints]` table (`clippy::all = deny`, `clippy::pedantic = warn`, `clippy::redundant_clone = deny`, `nonstandard_style = deny`).

The man page is authored in `man/findlargedir.1.scd` ([scdoc](https://git.sr.ht/~sircmpwn/scdoc) markup); the generated roff `man/findlargedir.1` is committed and bundled into release archives via `dist-workspace.toml`'s `include`. Its `OPTIONS`/`EXAMPLES` sections are hand-maintained, so **keep them in sync with `args.rs`'s `print_help` and the README** when adding or changing flags, then regenerate with the command above.

A sibling `AGENTS.md` carries the same guidance in condensed form; keep the two in sync when editing either.

## Architecture

`findlargedir` is a single-binary CLI tool that scans filesystems for "black hole" directories — directories with an extremely large number of entries that cause performance problems. It avoids doing expensive full `readdir` passes by using inode-size heuristics.

### Two-phase operation

**Phase 1 — Calibration (`src/calibrate.rs`)**
Creates empty files on the target filesystem in **geometrically growing** batches — the first batch is a floor of 1 000 (`-c` raises it) and each subsequent batch doubles — re-`stat`ing the temp directory after each batch, always sampling the full fixed schedule up to a 50 000-file cap (`FILE_CAP`). Geometric spacing makes the samples span the large-N range the ratio is later extrapolated onto, instead of clustering at low N. Two choices make the result **reproducible across runs** (an earlier adaptive early-stop and parallel creation made successive calibrations disagree): the schedule is fixed (no data-dependent early stop, which varied which regime got fit), and files are created **in order, not in parallel** (parallel insertion order jittered the htree layout, hence the per-`N` size, by a few percent). Files are named zero-padded to `calibration_name_length` (`-n`, default 24) so per-entry cost reflects representative entries rather than the minimal-name floor (which biased estimates high → false positives). A least-squares fit (`fit_calibration`) **over the upper-N half of the samples** gives the **asymptotic marginal** bytes-per-entry (slope) and **fixed overhead** (intercept); `fill_corrected` then divides the slope by `FILL_FACTOR` (0.75) because sequential calibration packs htree leaves tighter than real churned directories, which under-measures per-entry cost. The result is a `Calibration`. Fitting only the large-N window keeps the cheap first blocks (htree linear→hashed transition, block-size rounding) from skewing the slope used for million-entry directories. A filesystem whose large-N directory size never grows is detected (slope ≤ 0.5) and reported, with flagging disabled (`per_entry = 0`, the same sentinel as a shutdown mid-calibration). Calibration can be skipped with `-i <ratio>` (per-entry only, overhead 0) or pointed at a custom dir with `-t`. `classify_dir` guards against the zero-`per_entry` divide.

**Phase 2 — Parallel walk (`src/walk.rs`)**
Uses a custom `crossbeam-deque` work-stealing engine (`src/walk/engine.rs`, adapted from the sibling `minifind` project) to walk the filesystem in parallel, visiting directories only; a dedicated thread prints periodic progress (`-p`). The size estimate is split into two roles:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dkorunic/findlargedir](https://github.com/dkorunic/findlargedir) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
