---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

The project is managed with [`uv`](https://docs.astral.sh/uv/) and uses
the native `uv_build` backend — not hatch, setuptools, or poetry.

Most common commands are wrapped in [`Taskfile.yml`](./Taskfile.yml) — use
`task --list` to see them, or drive `uv` directly:

```bash
uv sync                         # install runtime + dev deps from uv.lock
uv run pytest                   # full test suite + coverage (fails < 75 %)
uv run pytest --no-cov tests/test_ols.py::test_ols_onedim  # a single test
uv run ruff check               # lint
uv run ruff format --check      # format-only check
uv run ty check                 # type check
uv run prek run --all-files     # pre-commit hooks (compatible with pre-commit)
uv build                        # emit sdist + wheel to ./dist
```

To exercise the Python-version matrix locally (all 3.10–3.14 are supported):

```bash
task test-matrix   # or: for V in 3.10 3.11 3.12 3.13 3.14; ...
```

Coverage is wired through `pytest-cov`; the 75 % floor is enforced by
`--cov-fail-under=75` in `[tool.pytest.ini_options]`.

## Before pushing: mandatory pre-push gauntlet

CI runs **four** gates that can independently fail, and `ruff check`
passing does **not** imply `ruff format --check` passes. Always run
all four locally before pushing, in this order:

```bash
uv run ruff format        # REWRITES files — run first, review the diff
uv run ruff check         # lint (separate from formatting)
uv run pytest             # full suite + coverage gate
uv run ty check           # type check (best-effort, not yet in CI)
```

Equivalent one-liner: `task check`.

Or, preferred: run the full pre-commit suite so you catch everything
CI checks plus markdownlint and the hygiene hooks:

```bash
uv run prek run --all-files
```

**Do not trust a passing `ruff check` to mean formatting is clean.**
Past regressions: appending new tests without a subsequent
`ruff format` run has silently broken CI's `ruff format --check` step
twice. The `pre-commit` hook also catches this; install with
`uv run prek install` to have it run on every commit.

## High-level architecture

Six modules in `src/pyfinance/`. Public re-exports live in
`pyfinance/__init__.py`; currently only `TSeries` and `TFrame` are
surfaced at the top level.

### `returns.py` — subclassed pandas

`TSeries` and `TFrame` **subclass `pd.Series` and `pd.DataFrame`** and
add the performance-statistics surface (annualized return/vol, drawdown,
Sharpe, capture ratios, CAPM alpha/beta).

Subclassing pandas is delicate. The `_constructor` / `_constructor_expanddim`
properties are what keep operations returning a `TSeries` rather than a
plain `Series` — don't remove them. `TFrame.__init__` currently does not
pass through to `pd.DataFrame.__init__`, so `TFrame(...)` cannot be
constructed like a normal DataFrame; instances arise from operations on
a `TSeries` that dispatch through `_constructor_expanddim`. This is
latent technical debt, not a bug to fix casually.

### `ols.py` — NumPy-backed regression

`OLS` is the static case; `RollingOLS` is the 3-D matrix formulation
(rolling via `.swapaxes(1, 2)` tricks); `PandasRollingOLS` is a thin
Pandas-indexed wrapper.

Most result properties are `@cached_property` — **not** `@property +
@functools.cache` (the latter leaks references to `self`). Preserve this
when adding new result properties.

### `options.py` — vectorized BSM

`BSM` for valuation, Greeks, implied vol; inheritance hierarchy of
strategy classes (`Straddle`, `BullSpread`, `_Butterfly`, `_Condor`,
etc.) underneath `Option` / `OpStrat`.

`kind: Literal["call", "put"]` is the one narrowly-typed choice here;
keep the `Literal` in any signature that proxies to it.

### `general.py` — heterogeneous numerical tools

Active share, amortization schedules, best-fit-distribution search, PCA
on returns, portfolio simulation, tracking-error optimization, VIF.

`factor_loadings` **requires** a `factors` DataFrame from the caller.
The old implicit-fallback to `datasets.load_factors` was removed in
2.0, along with the loader itself.

### `datasets.py` — kept lean after 2.0

Public surface is `load_13f`, `load_industries`, `load_rates`, `load_rf`.
Two details matter:

- **Lazy `pandas_datareader` proxy.** `datasets.pdr` is a
  `_LazyPandasDataReader` instance, *not* the real module. This exists
  because `pandas_datareader` 0.10.0 imports the stdlib `distutils`
  module, which was removed in Python 3.12. The proxy defers the import
  to attribute-access time, so `import pyfinance.datasets` works on
  modern Pythons. Attribute-style use (`pdr.DataReader(...)`) continues
  to work; module-identity checks (`isinstance(pdr, ModuleType)`) will
  not.
- **`load_13f` requires a `user_agent` string.** SEC EDGAR refuses
  requests without a descriptive User-Agent. `load_13f` raises
  `ValueError` before making a request if the argument is empty.

### `utils.py` — frequency and array helpers

Frequency conversion, rolling-window construction, one-hot encoding,
random ticker/weights generation.

The `PERIODS_PER_YEAR` dict is keyed by `FreqGroup.<X>.value` (not the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bsolomon1124/pyfinance](https://github.com/bsolomon1124/pyfinance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
