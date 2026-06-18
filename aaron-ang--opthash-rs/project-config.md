---
trigger: always_on
description: Use this file as the local operating guide for code changes in this repo.
---

# AGENTS.md

Use this file as the local operating guide for code changes in this repo.

## Commands

For code refactors, run:

```bash
cargo test                                  # Run all tests
pre-commit run --all-files                  # Runs formatters, clippy, ruff, mypy stub checks
```

For performance-sensitive changes, run the relevant benchmark A/B workflow in [Benchmarks](#benchmarks) and read the JSON results.

One-time setup (after cloning):

```bash
uv tool install pre-commit
pre-commit install
```

## Benchmarks

Criterion suite comparing `ElasticHashMap`, `FunnelHashMap`, `std::HashMap`, `hashbrown::HashMap` (SwissTable + foldhash — absolute ceiling).

Use `scripts/bench.sh` for benchmark results you will act on. Raw `cargo bench` is unpinned; wall-clock noise can swing and flip the sign of real changes. Use raw cargo only for smoke runs or single-filter iteration.

For A/B comparisons, save a known anchor, save each changed tree as a named variant, then compare stored runs offline with `LOAD` and `BASELINE`. Refresh `ref` when intentionally updating the default anchor after environment or benchmark-fixture changes.

```bash
SAVE=anchor scripts/bench.sh                # measure known baseline
SAVE=opt1 scripts/bench.sh                  # measure changed tree
LOAD=opt1 BASELINE=anchor scripts/bench.sh  # opt1 vs anchor (no rerun)
SAVE=ref scripts/bench.sh                   # intentionally refresh the default anchor
```

For many variants against the same anchor, save the anchor once (`SAVE=anchor`), save each variant (`SAVE=optN`), then compare with `LOAD=optN BASELINE=anchor`. Stored baselines persist in `target/criterion/`.

- `target/criterion/<group>/<variant>/new/estimates.json` — absolute ns (`mean.point_estimate`)
- `target/criterion/<group>/<variant>/change/estimates.json` — fractional change vs the selected baseline (`+0.05` = 5% slower)
- Variants are `<op>_<impl>` per [benches/speedup.rs](benches/speedup.rs), for example `get_hit_funnel` or `insert_elastic`.
- Criterion group dir = `<workload>`; see [benches/README.md](benches/README.md) for the full id convention.
- Example change file: `target/criterion/get_hit/get_hit_funnel/change/estimates.json`.

Run shape and options:

- Wraps `cargo bench` with `taskset` (core pin), `setarch -R` (ASLR off), `chrt -b` (scheduler batch), and `numactl` (NUMA bind, multi-node only) — no privileges. `sudo` adds `nice -20`, `prlimit` memlock; drops back to invoking user for cargo.
- `BENCH=all` (default) runs `speedup`, then `mean_latency`; set `BENCH=speedup|mean_latency` for one target.
- Other `[[bench]]` targets are not in `all`; run explicitly via `BENCH=<name>`: `set_ops` (set algebra), `map_api` (entry/iter/drain/extract_if), `load_factor`, `payload_size`.
- Pass through Criterion flags with `--`: `SAVE=opt1 scripts/bench.sh -- --measurement-time 10`.
- Filter by Criterion name: `SAVE=opt1 scripts/bench.sh -- "get_hit_latency"`. The script strips a leading `--`, so both forms work.

### Latency-chart harness

- `BENCH=mean_latency scripts/bench.sh` — Criterion sweep of `get_hit` over `LATENCY_SIZES` (1K → 10M); feeds the cache-cliff line chart. Output: `target/criterion/get_hit_latency_<size>/get_hit_latency_<size>_<impl>/`.

### Python-side benchmarks

`benches/python/throughput.py` — pytest-benchmark suite comparing `dict`, `ElasticHashMap`, and `FunnelHashMap` from Python across insert / get_hit / get_miss / mixed / delete workloads at N = 10K. Each opthash op crosses the GIL → `HashedAny::hash()` → Python bytecode.

```bash
pytest benches/python/throughput.py --benchmark-json=.benchmarks/python.json

uv run scripts/generate_python_chart.py
```

### CodSpeed CI

`.github/workflows/codspeed.yml` runs Callgrind sim on every PR. Two jobs:

- **rust** — `cargo codspeed run --bench speedup`. The `criterion` dev-dep is a package rename to `codspeed-criterion-compat`; don't revert.
- **python** — `pytest benches/python/throughput.py --codspeed`. `pytest-codspeed` is drop-in for `pytest-benchmark`.

`mean_latency.rs` is local-only. Sim counts instructions, not wallclock — `scripts/bench.sh` stays the local ground truth.

### Charts

- `uv run scripts/generate_speedup_chart.py` — throughput speedup bar chart
- `uv run scripts/generate_latency_chart.py` — Criterion mean-latency line (`target/criterion/get_hit_latency_<size>`; sizes from `LATENCY_SIZES` in `benches/common.rs`).
- `uv run scripts/generate_all_charts.py` — regenerate everything
- `uv run scripts/generate_python_chart.py` — Python-side dict-vs-opthash speedup (reads `.benchmarks/python.json`)

Charts are saved in `assets/`. Shared plotting helpers (`IMPLEMENTATIONS`, loaders, axis styling) live in `scripts/_plot_common.py`.

### Methodology

- Re-save the anchor whenever a fixture constant changes (`OP_COUNT`, `MAP_SIZE`, `LATENCY_SIZES`). Comparing across different workloads makes `change/` deltas meaningless.
- Treat unchanged controls (`std`, `hashbrown`) as the run's noise floor. Large movement in controls weakens any conclusion about `elastic` or `funnel`.
- Rebuild before reading `callgrind`/CodSpeed output — stale binaries silently report pre-change asm. Check binary mtime against the commit you intend to measure.

## Project Structure


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aaron-ang/opthash-rs](https://github.com/aaron-ang/opthash-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
