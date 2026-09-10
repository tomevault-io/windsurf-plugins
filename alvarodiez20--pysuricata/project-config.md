---
trigger: always_on
description: Streaming EDA profiler. Single pass over the data, bounded memory, emits a
---

# PySuricata — working notes for Claude Code

Streaming EDA profiler. Single pass over the data, bounded memory, emits a
self-contained HTML report. pandas + polars native.

## Commands

```bash
uv sync --dev
uv run pytest -m "not benchmark"            # test suite
uv run pytest benchmarks/accuracy.py -v     # statistical accuracy oracle
uv run ruff check . && uv run ruff format . # lint (line-length 88, py310 target)
uv run mkdocs serve                         # docs
uv run python -m benchmarks.check_docs --strict   # docs + README vs the live API

# Layout acceptance criteria in a real browser (#124). Not in `dev`: Chromium is
# ~300 MB and only these 31 cases need it, so they skip when it is absent.
uv sync --all-extras --group browser && uv run playwright install chromium
uv run pytest -m browser
uv run python scripts/contact_sheet.py      # 6 review captures, never a gate

# Real Pyodide boot + profile run, asserted on rendered pixels not markup (#1).
# Slow and network-dependent (PyPI, jsDelivr) -- not part of `pytest -m browser`,
# runs post-release in cd.yml's demo-check job against the live site instead.
uv run python web/e2e.py                                    # local web/, over loopback
uv run python web/e2e.py --url https://pysuricata.pages.dev # the live demo

python -m benchmarks.hotspots               # where does profile() spend its time
python -m benchmarks.kernels                # per-kernel timings + memory roofline
python -m benchmarks.end_to_end --markdown results.md   # vs ydata/sweetviz/skimpy

cargo test --lib --manifest-path native/Cargo.toml      # native kernels
maturin develop --release -m native/Cargo.toml          # build + install locally
```

## Architecture

```
pysuricata/
  api.py              profile() / summarize() — public surface
  config.py           ProfileConfig; ComputeOptions is the user-facing knob set
  compute/
    orchestration/    engine.py — the chunk loop, adapter dispatch, checkpointing
    adapters/         pandas.py, polars.py — frame-shaped I/O
    processing/       chunking.py (chunk sizing), inference.py (column typing)
    analysis/         correlation.py
    consume.py        pandas chunk -> accumulator wiring
    consume_polars.py polars equivalent
  accumulators/       the statistical core — numeric, categorical, datetime, boolean
    algorithms.py     StreamingMoments (Welford/Pébay), ExtremeTracker, monotonicity
    sketches.py       KMV distinct-count, MisraGries top-k, ReservoirSampler, RowKMV
  render/             HTML generation; html.py is the template driver
  templates/, static/ report shell, CSS, JS
native/               optional Rust kernels (pysuricata-core, PyO3 + maturin)
benchmarks/           accuracy oracle + performance harness
```

Data flows one way: adapter yields chunks -> `consume_chunk_*` converts each
column to an array -> the matching accumulator's `update()` folds it in ->
`finalize()` produces a summary dataclass -> `render/` turns summaries into HTML.
Accumulators never see the frame, only arrays.

## Conventions

- Accumulators must be **mergeable** and **order-independent** where the statistic
  allows it. Chunked results must equal unchunked results; that invariant is
  asserted in `benchmarks/accuracy.py` and is the thing most likely to break.
- Approximate values must be labelled approximate. Sketches carry error bounds;
  surface them rather than printing a sketch estimate as an exact integer.
- Never touch the global RNG. Seeds belong to the accumulator instance.
- The pure-Python path is the reference implementation. The native crate is an
  optional accelerator and must agree with it within documented tolerance —
  never delete the Python path to "simplify".
- Ruff, line length 88, `from __future__ import annotations` at the top of modules.

## Current priorities

**The issue tracker is the authority.** `docs/roadmap.md` was v10, pinned to
0.0.62, and shipped in the docs nav describing a project ninety releases older
than the one readers were installing; it is deleted rather than re-synced,
because a roadmap in the docs dates the moment it is written and nothing after.
The working roadmap is **v15** and lives outside the repo. Half of #251 goes
with the file; what is left of it is the per-column figure, now measured.

**0.1.5 is published.** The report redesign is closed out, and so are
both packaging blockers, the README drift, the per-chunk missing counts and the
Missing-pane gate. Do not regress any of it — a change that makes
`benchmarks/accuracy.py` fail is wrong even if it is faster, and the same goes
for `tests/test_report_data_invariance.py`.

Three ratchets now guard things that only go one way. Each fails **in both
directions**: growth is a regression, and shrinking asks you to lower the
baseline so the win cannot be quietly respent.

| ratchet | where |
|---|---|
| report bytes, and elements per card | `tests/test_report_layout.py` |
| untokenised colours | `tests/test_colour_tokens.py` |
| `Processed bytes` still in a stat row | `tests/test_processed_bytes_placement.py` |

Next, in rough order. **Check state before starting** — this list has gone
stale twice, and #306 was worked on after it was already delivered:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alvarodiez20/pysuricata](https://github.com/alvarodiez20/pysuricata) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
