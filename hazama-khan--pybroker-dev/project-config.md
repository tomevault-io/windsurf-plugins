---
trigger: always_on
description: validates the `cache=True` contract. Never add warmup to it.
---

# PyBroker — Core Library Guide

PyBroker is a Python framework for developing and backtesting algorithmic
trading strategies, with a focus on strategies driven by machine learning.
Its backtesting engine is built on NumPy and accelerated with Numba. Users
define per-bar execution functions that place orders through an
`ExecContext` (`Strategy.add_execution`); the engine replays historical
bars for multiple instruments, simulates fills with fees, slippage, stops,
and position limits through a `Decimal`-based `Portfolio`, and reports
results as `TestResult` DataFrames with bootstrapped confidence intervals
on the metrics. Models are trained and evaluated with Walkforward
Analysis: the data is split into successive train/test windows so models
only ever predict on bars that came after their training data. On top of
that sit user-defined and built-in indicators, hyperparameter optimization
(Optuna), multi-timeframe intervals, rotational trading, ranked position
sizing, disk caching of data/indicators/models, and parallelized
computation. Bar data comes from built-in data sources (Alpaca, Yahoo
Finance, AKShare) or any user-supplied DataFrame/`DataSource`.

Import name `pybroker`, PyPI name `lib-pybroker`. Version is
single-sourced at `src/pybroker/__init__.py:__version__` (setup.cfg reads
it via `attr:`). Work integrates on `dev`; PRs target `dev`, and `master`
is the release branch. This
file governs changes to the core library (`src/pybroker/`) and the
distributable agent skills (`skills/`).

## Commands

```bash
# Setup (once): Python 3.11+ venv, then editable install with test deps
pip install -e ".[test]"

# Tests (~5,000). Local venvs are gitignored (.venv*) — use a project venv
# on the tooling Python (3.12) if the checkout has one, e.g.
# .venv-bench/bin/python -m pytest.
python -m pytest                           # full suite
python -m pytest tests/test_<module>.py    # one module (mostly 1:1 with src)
python -m pytest -n auto --dist loadgroup  # parallel; keeps xdist_group pins (ray/loky)
python -m pytest -p no:randomly ...        # deterministic order when bisecting failures

# Quality gates (tox envs defined in setup.cfg)
tox -e format             # ruff format --diff — CHECK ONLY; `tox -e format -- src tests` to write
tox -e lint               # ruff check src tests
tox -e typecheck          # mypy on src (mypy version pinned in the tox env)
tox -e py311,py312,py313,py314  # full test matrix

# Benchmarks (asv; see Performance & Benchmarks)
asv run --quick             # fast feedback, one sample per benchmark
# what the CI PR gate measures with; the block/pass decision itself is made
# by .github/scripts/asv_gate.py (blocks at 1.25x, but only above a 10ms
# baseline — see Performance & Benchmarks)
asv continuous dev HEAD --factor 1.1 --interleave-rounds

# Docs — CI runs `tox -e docs` on Python 3.12 (`[testenv:docs] basepython`);
# it is STRICT (`sphinx-build -n -W --keep-going`), so any warning fails the
# build. Deps come from requirements.txt. Run the same flags from a project
# venv on 3.12 instead of a bare `sphinx-build -b html`, which hides
# warnings that fail CI.
python -m sphinx -n -W --keep-going -b html docs/source/ docs/_build/
```

## Iron Rules

1. **Never leak future bars.** No negative indexing or backward shifts that
   read past the current bar; new indicators must join the no-lookahead
   sweep. (§ Lookahead-Bias Guardrails)
2. **Pandas only at the I/O boundary.** Core computation is NumPy + Numba;
   six modules are pandas-free and must stay that way. (§ Pandas Boundary)
3. **Every compiled kernel is `@njit(cache=True)`.** 87/87 today; zero
   exceptions. (§ NumPy + Numba Core)
4. **Never widen, mutate, or copy the user's input DataFrame.** Feature
   data stays numpy-backed and out-of-band. (§ Project Rules)
5. **Do not "clean up" mid-file or lazy imports.** They break intentional
   cycles; E402 is disabled in ruff for exactly this. (§ Architecture)
6. **Decimal for money and share counts; float64 for everything
   vectorized.** Quantize to cents only at the output boundary. (§ Money,
   Floats & Determinism)
7. **Identical results every run.** Preserve every determinism rationale
   comment; never iterate an unsorted set into results. (§ Money, Floats &
   Determinism)
8. **Never `git stash`; commit/push only when asked.** Use a detached
   worktree for comparisons. (§ Project Rules)

## Architecture & Layering

The layering ladder below is also the module inventory of `src/pybroker/`:

```
L0  common, vect, parallel   — import nothing from pybroker at runtime
L1  interval, log, config    — import common only
L2  scope                    — common, interval, log
L3  cache, portfolio, eval, slippage, data
L4  model, indicator, optimize
L5  context
L6  strategy                 — the only module that may import everything
    ext/data.py              — common + data only (optional data sources)
```

- **Runtime imports point downward only.** If a change needs an upward
  import, the code is in the wrong module — move it, don't import it.
  (`common.py` has one `TYPE_CHECKING`-only `from pybroker.strategy import
  Execution`, which doesn't count — it never executes.)
- **Intentional cycle breaks — do not "fix" (as of 2.0.0):**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hazama-khan/pybroker_dev](https://github.com/hazama-khan/pybroker_dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
