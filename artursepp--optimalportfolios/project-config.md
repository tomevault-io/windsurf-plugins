---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**OptimalPortfolios** (v4.1.1) is a Python library for constructing and backtesting multi-asset portfolios. It provides the full production pipeline: alpha signal computation → covariance estimation (EWMA or HCGL factor model) → portfolio optimisation (risk budgeting, max diversification, max Sharpe, alpha-over-tracking-error, etc.) → rolling backtest with NaN-aware data handling.

The package is the reference implementation for the ROSAA framework published in *The Journal of Portfolio Management* (Sepp, Ossa, Kastenholz, 2026).

## Commands

### Install (editable with dev dependencies)
```bash
pip install -e ".[dev]"
```

### Run all tests
```bash
pytest optimalportfolios/
```

### Run a single test file
```bash
pytest optimalportfolios/alphas/tests/signals_test.py -v
pytest optimalportfolios/optimization/tests/constraints_test.py -v
```

### Formatting and linting
```bash
black optimalportfolios/       # Format (line-length=100)
isort optimalportfolios/       # Sort imports (profile=black)
flake8 optimalportfolios/      # Lint
mypy optimalportfolios/        # Type-check
```

Test files follow `*_test.py` naming; pytest markers include `slow`, `integration`, `unit`, `optimization`, `backtesting`.

## Architecture

### Package Structure
```
optimalportfolios/
├── alphas/                    # Alpha signal computation (NEW in v4.1.1)
│   ├── signals/
│   │   ├── momentum.py        # compute_momentum_alpha()
│   │   ├── low_beta.py        # compute_low_beta_alpha()
│   │   └── managers_alpha.py  # compute_managers_alpha()
│   ├── alpha_data.py          # AlphasData container
│   ├── backtest_alphas.py     # Signal backtesting tool
│   └── tests/
├── covar_estimation/          # Covariance matrix estimation
│   ├── covar_estimator.py     # CovarEstimator ABC
│   ├── ewma_covar_estimator.py    # EwmaCovarEstimator
│   ├── factor_covar_estimator.py  # FactorCovarEstimator (HCGL)
│   ├── rolling_covar.py       # RollingFactorCovarData, CurrentFactorCovarData
│   └── covar_reporting.py     # Rolling covariance diagnostics
├── lasso/                     # HCGL factor model
│   └── lasso_model_estimator.py
├── optimization/              # Portfolio optimisation
│   ├── constraints.py         # Constraints, GroupLowerUpperConstraints
│   ├── wrapper_rolling_portfolios.py  # compute_rolling_optimal_weights()
│   └── solvers/               # One module per solver, each with 3 layers
├── utils/                     # Auxiliary analytics
├── reports/                   # Performance reporting
└── examples/                  # Worked examples and paper reproductions
```

### Module Dependency Order
```
alphas/  →  optimization/  →  covar_estimation/  →  lasso/  →  utils/  →  reports/
```
`alphas/` depends only on `qis` and standard libraries. `optimization/` depends on `covar_estimation/` only for type hints — covariance estimation is separated from optimisation (covar_dict is passed, not estimated internally).

### Key Design Principle: Estimation/Optimisation Separation

Covariance estimation is separated from portfolio optimisation. Estimate first, then pass `covar_dict` to any solver:

```python
# estimate once
estimator = EwmaCovarEstimator(returns_freq='W-WED', span=52, rebalancing_freq='QE')
covar_dict = estimator.fit_rolling_covars(prices=prices, time_period=time_period)

# reuse across multiple taa
weights_rb = rolling_risk_budgeting(prices=prices, covar_dict=covar_dict, ...)
weights_md = rolling_maximise_diversification(prices=prices, covar_dict=covar_dict, ...)
```

Rolling solvers do NOT estimate covariance internally — `covar_dict` is a required parameter.

### Three-Layer Solver Pattern

Every portfolio solver is implemented in three layers. This pattern MUST be followed when adding new solvers:

| Layer | Prefix | Input | Output |
|-------|--------|-------|--------|
| 1 – Math | `opt_*` / `cvx_*` | Clean `np.ndarray`, no NaNs | `np.ndarray` weights |
| 2 – Wrapper | `wrapper_*` | `pd.DataFrame` (may have NaNs) | `pd.Series` weights |
| 3 – Rolling | `rolling_*` / `backtest_*` | `pd.DataFrame` prices + `covar_dict` | `pd.DataFrame` weight time series |

The wrapper layer filters NaN assets via `filter_covar_and_vectors_for_nans()`, calls `constraints.update_with_valid_tickers()`, then delegates to layer 1. The rolling layer iterates over rebalancing dates in `covar_dict` and calls the wrapper.

### Alpha Signals Module (v4.1.1)

Three standalone signal functions with a consistent interface:

```python
def compute_*_alpha(
    prices: pd.DataFrame,
    returns_freq: Union[str, pd.Series],  # single or mixed frequency
    group_data: Optional[pd.Series],      # for within-group scoring
    **signal_params,
) -> Tuple[pd.DataFrame, pd.DataFrame]:   # (score, raw_signal)
```

Each function handles both single-frequency (`returns_freq='ME'`) and mixed-frequency (`returns_freq=pd.Series(...)`) internally. No separate `_different_freqs` variants.

Naming convention: raw signal → score → alpha.

`AlphasData` is the output container holding `alpha_scores` (portfolio-ready) plus all intermediate components for diagnostics.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ArturSepp/OptimalPortfolios](https://github.com/ArturSepp/OptimalPortfolios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
