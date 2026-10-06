---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Benchmark companion code for this project on **causal foundation models** — zero-shot, amortised CATE estimators trained via PFN-style meta-learning. Compare three foundation models against six traditional metalearners on synthetic and real-world data.

### Foundation Models

| Model | Install method |
|---|---|
| **CausalPFN** | Core dependency, `causalpfn>=0.1.5` from PyPI (0.1.5 is the first release with the Apple Silicon and `estimate_ate_CI` fixes); weights downloaded from HF Hub on first use |
| **Do-PFN** | Clone `jr2021/Do-PFN`, add to `sys.path` — not its `requirements.txt` as-is, see caveats below |
| **CausalFM** | Clone `yccm/CausalFM-toolkit`, install requirements, provide a checkpoint path |

### Traditional Metalearners (from econml)

| Method | Type | Key feature |
|---|---|---|
| **S-learner** | Single-model | Trains one model on X⊕T |
| **T-learner** | Two-model | Separate models for T=0, T=1 |
| **X-learner** | Cross-fitting | Asymptotically efficient variant |
| **Debiased ML** | Neyman-orthogonal | Robust to nuisance parameter estimation |
| **IPW** | Inverse Probability Weighting | Based on propensity scores |
| **DR** | Doubly Robust | Combines outcome + propensity modeling |

## Commands

### Local setup (with uv)

```bash
uv sync --all-extras              # core + metalearners + causalfm deps
jupyter notebook notebooks/Foundation_models_quickstart.ipynb   # start exploring
```

Or with pip:

```bash
pip install -r requirements.txt   # core deps
jupyter notebook notebooks/Foundation_models_quickstart.ipynb
```

There's no dedicated setup/data notebook — synthetic dataset generation is inline via `causal_bench.data_generators` (see the smoke-test below), used directly wherever a notebook needs a dataset.

### Running notebooks

Three notebooks, no numeric ordering — pick based on what you need. (The Lalonde benchmark *run itself* used to be a notebook; it's now `scripts/run_benchmark.py` — see the next section. `Lalonde_benchmark_results.ipynb` below only reads that script's already-committed output, it doesn't run the benchmark.)

**1. Foundation Models Quickstart** (fastest path to one working model):
```bash
jupyter notebook notebooks/Foundation_models_quickstart.ipynb
```
- Covers **CausalPFN** alone, end to end: install, fit, predict. Standalone — calls CausalPFN's native API directly, not this repo's wrapper classes, so any cell can be copied into another project as-is.
- Deliberately narrow (one model, one dataset) — see the sandbox notebook below for a three-model comparison.

**2. Foundation Models Sandbox** (practitioner playground, no `causal_bench` wrappers):
```bash
jupyter notebook notebooks/Foundation_models_sandbox.ipynb
```
- **A sandbox/playground notebook, not a quickstart** — per advisor feedback, a real "quickstart" should cover a single model (CausalPFN) end to end; that's what notebook 1 above is for. This notebook instead runs all three foundation models side by side and is checked into git specifically to preserve the observations, caveats, and gotchas found while exploring them together.
- Standalone — calls each library's own native API directly (`CATEEstimator`/`ATEEstimator`, `DoPFNRegressor`, `StandardCATEModel`), not this repo's wrapper classes
- Simulates one self-contained example dataset inline (a confounded discount-email scenario with known heterogeneous CATE) instead of using `causal_bench.data_generators`
- One cell per model, each with markdown explaining install/setup and any library-specific gotchas

**3. Lalonde Benchmark Results** (loads and visualizes `scripts/run_benchmark.py`'s output):
```bash
jupyter notebook notebooks/Lalonde_benchmark_results.ipynb
```
- Reads the production run's already-committed results — `data/benchmark_results_cpu.csv` (all 9 models) and `data/benchmark_results_gpu.csv` (the 3 foundation models, re-run on GPU) — no benchmark run required to use this notebook
- Averages over the 10 realizations per cohort into a summary table (PEHE in units of $1,000 to match CausalPFN's Table 1 units, plus per-cohort and pooled PEHE/ATE-relative-error ranks), and a rank-vs-runtime figure styled after CausalPFN's own paper Figure 1 (foundation models vs. metalearners vs. Debiased ML called out separately, since it's the best-ranked non-foundation model)
- Deliberately does **not** normalize runtime "per 1,000 samples" the way the paper's figure axis is labeled: the metalearners' HPO time is a fixed 900s *budget* per sub-model regardless of dataset size (verified directly: CPS has ~6x PSID's sample count but near-identical metalearner runtimes), so normalizing by sample count would make PSID look artificially cheaper for a cost that's actually constant. Raw per-task CPU seconds is used instead.
- Uses `causal_bench` wrappers only insofar as the CSVs it reads were produced by them — the notebook itself is pure pandas/matplotlib over the CSVs, no model libraries imported

### Running the benchmark script

**`scripts/run_benchmark.py`** is the canonical Lalonde benchmark — compare all 3 foundation models vs. all 6 metalearners:
```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [layer6ai-labs/cfms](https://github.com/layer6ai-labs/cfms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
