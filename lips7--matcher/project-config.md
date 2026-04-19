---
trigger: always_on
description: High-performance multi-language word/text matcher in Rust with Python, C, and Java bindings. Solves precision/recall problems in pattern matching via logical operators (`&`/`~`/`|`/`\b`) and text normalization pipelines.
---

# Project

High-performance multi-language word/text matcher in Rust with Python, C, and Java bindings. Solves precision/recall problems in pattern matching via logical operators (`&`/`~`/`|`/`\b`) and text normalization pipelines.

**Toolchain:** nightly Rust (see `rust-toolchain.toml`), edition 2024. Nightly is required — do not change this. Edition 2024 has behavioral changes: `gen` is reserved, `unsafe` blocks required inside `unsafe fn`, etc.

**Prerequisites:** `just` (command runner), `cargo-all-features` (`cargo install cargo-all-features`), `cargo-nextest` (`cargo install cargo-nextest`), `uv` (Python env manager), `prek` (pre-commit runner).

## Commands

```bash
# Build
just build                          # Full workspace + copy bindings artifacts
cargo build --release               # Rust only

# Quick iteration
just check                          # Fast type-check (no codegen)
just fmt                            # Auto-format
just fmt-check                      # Check formatting without modifying

# Test (test-rs and test-quick accept pass-through args)
just test                                          # All languages (Rust + Python + Java + C)
just test-rs                                       # All feature combos + doctests + docs
just test-quick                                    # Default-features tests only
just test-quick test_name                          # Single test by name (substring match)
just test-quick --no-default-features              # Without DFA
just test-quick --test test_engine                 # Single test file by name
just test-py                                       # Python bindings
just test-java                                     # Java bindings
just test-c                                        # C bindings

# Lint/Format
just lint                           # All languages + workspace clippy + doc build
just lint-check                     # Check-only (no auto-fix) — used in CI
just lint-rs                        # cargo fmt + cargo clippy (matcher_rs)
just lint-py                        # cargo fmt + cargo clippy + ruff + ty check (matcher_py)
just lint-java                      # cargo fmt + cargo clippy + mvn checkstyle (matcher_java)
just lint-c                         # cargo fmt + cargo clippy (matcher_c)

# Benchmarks (harness: divan, single target: bench)
# All bench recipes accept pass-through args: --quick, --profile, --repeats, --filter, etc.
just bench-search                                      # Main throughput workflow (~15 min)
just bench-search --quick                              # Quick directional signal (~2-3 min)
just bench-search --filter text_transform              # Only transform benchmarks (~2 min)
just bench-search --filter rule_complexity             # Only rule shape benchmarks (~3 min)
just bench-search --filter "scaling::process_cn"       # Single benchmark group (~1 min)
just bench-search --profile bench-dev                  # Faster rebuild (thin LTO)
just bench-build                                       # Matcher construction workflow
just bench-all                                         # All presets (search + build)
just bench-compare <baseline> <candidate>               # compare runs, dirs, or raw files
just bench-viz <run_dir>                               # interactive HTML dashboard (Plotly)
just bench-viz <baseline_dir> <candidate_dir>          # comparison visualization

# Profiling (scene-based, uses release + debug symbols)
cargo run --profile profiling --example profile_search -p matcher_rs -- --list        # list scenes
cargo run --profile profiling --example profile_search -p matcher_rs -- --scene all   # all scenes
just profile record --scene en-search --analyze        # Instruments + auto-analyze

# Coverage
just coverage                       # cargo tarpaulin → matcher_rs/tarpaulin-report.html

# Dependency updates
just update                         # cargo update --breaking + cargo upgrade

# Release
scripts/bump-version.sh <version>   # Update version in all manifests + CHANGELOG
```

**Pre-commit:** `.pre-commit-config.yaml` exists — run `prek run` before committing.

**Cargo profiles:** `bench` (full LTO + debug symbols — authoritative measurements via `run_benchmarks.py`), `bench-dev` (thin LTO + incremental — faster rebuild for iterative bench development), `profiling` (release + debug symbols — for `instruments`/`perf`/`samply`).

## Architecture

For the full narrative walkthrough with a running example, see [DESIGN.md](./DESIGN.md). Below is the essential mental model.

### How a Query Works

1. **Transform** — Walk a shared-prefix trie of `ProcessType` steps, producing text variants (VariantNorm, Delete, Normalize, Romanize, RomanizeChar, EmojiNorm). Intermediate results are reused across combinations.
2. **Scan** — Each variant is scanned by a single deduplicated Aho-Corasick automaton (bytewise or charwise, selected by SIMD density scan at threshold 0.67). Hits update per-rule state.
3. **Evaluate** — Touched rules are checked: all AND segments satisfied + no NOT veto → match.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Lips7) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
