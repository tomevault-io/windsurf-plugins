---
trigger: always_on
description: Guidance for AI coding agents working in the **OptimalPortfolios** repository.
---

# AGENTS.md

Guidance for AI coding agents working in the **OptimalPortfolios** repository.

## Project overview

`optimalportfolios` implements the production pipeline for multi-asset portfolio construction and backtesting: alpha signals -> covariance estimation (EWMA or the HCGL factor model from `factorlasso`) -> constrained optimisation (risk budgeting, maximum diversification, maximum Sharpe, alpha over tracking error, and others) -> rolling backtest and reporting through `qis`.

It is the reference implementation of the ROSAA framework published in *The Journal of Portfolio Management* (Sepp, Ossa and Kastenholz, 2026). Distribution and import name `optimalportfolios`. Licensed MIT (`LICENSE.txt`).

## Ecosystem position

This package is one of eight open-source Python libraries maintained at [github.com/ArturSepp](https://github.com/ArturSepp). Before implementing anything non-trivial, check whether it already exists in one of these:

| Package | Repository | Purpose |
|---|---|---|
| `qis` | QuantInvestStrats | Performance analytics, factsheets, visualisation |
| `optimalportfolios` | OptimalPortfolios | Portfolio construction and backtesting |
| `factorlasso` | factorlasso | Sparse factor models and factor covariance estimation |
| `bbg-fetch` | BloombergFetch | Bloomberg data fetching |
| `trendfollowing` | TrendFollowingSystems | Trend-following systems: closed-form theory and replication |
| `goal-based-allocation` | GoalBasedAllocation | Dynamic MV allocation under regime-switching jump-diffusions |
| `stochvolmodels` | StochVolModels | Stochastic volatility pricing analytics |
| `vanilla-option-pricers` | VanillaOptionPricers | Vanilla option pricers and implied volatility fitters |

Actual package dependencies within the stack: `optimalportfolios` depends on `qis` and `factorlasso`; `trendfollowing` depends on `qis`; `stochvolmodels` has an optional `research` extra that pulls in `qis`. The others are independent.

Do not vendor or copy code between these packages. If functionality belongs in a sibling package, say so rather than reimplementing it here.

### `rosaa` dependency floors

`rosaa/` is gitignored and carries no `pyproject.toml`, so its floors have nowhere else to live and are recorded here. They are not advisory: each names a symbol or keyword `rosaa` calls that does not exist below the floor.

| Package | Floor | What `rosaa` needs at it |
|---|---|---|
| `qis` | **>= 5.5.0** | `load_df_from_csv` / `load_df_dict_from_csv` take `float_precision`; the inputs store cannot round-trip a float exactly without it |
| `factorlasso` | **>= 0.11.0** | `RollingFactorCovarData.get_alphas` forwards `asset_frequencies` / `default_freq`; below it a per-frequency `alpha_span` silently applies the `'ME'` entry to every quarterly asset |
| `optimalportfolios` | **>= 6.8.0** | signal spans accept a per-cadence `Mapping[str, int]`; below it `product_config.SIGNALS` raises, since it passes dicts |

`optimalportfolios 6.7.0` was tagged in `CITATION.cff` but never published — its `pyproject.toml` stayed at 6.6.0 — so a fresh `pip install optimalportfolios` before 6.8.0 gives a package `rosaa` cannot run on. Verified with `pip index versions optimalportfolios`, not from the changelog.

## Repository layout

```
optimalportfolios/
  alphas/            alpha signal construction
  covar_estimation/  covariance estimators (EWMA, factor/HCGL via factorlasso)
  optimization/      optimisers, constraints, solvers
  universe/          instrument universes
  reports/           reporting built on qis
  tests/             cross-cutting tests (release metadata agreement)
  utils/, examples/, docs/, config.py, local_path.py, settings.yaml
papers/              code accompanying the published papers (excluded from ruff)
```

Tests live inside the package as `optimalportfolios/<subpackage>/tests/*_test.py`; there is no top-level `tests/` directory. Not every `*_test.py` is a pytest module: sixteen of them are `run_local_test` diagnostic scripts that print and plot, contribute no collected tests, and need the author's local price data. They are still imported during collection, so they must stay importable on a core install — put an optional import inside the function that needs it and raise `ImportError` naming the extra.

## Commands

```bash
pip install -e ".[dev]"                                  # editable install with dev tools
pytest                                                   # run the test suite (180 tests, ~9 s)
pytest optimalportfolios/optimization/tests/constraints_test.py -v
ruff check optimalportfolios/                            # lint (papers/ is excluded)
```

*Note: Terminal execution should be compatible with Windows PowerShell within PyCharm.*

Optional extras: `data`, `reports`, `visualization`, `jupyter`, `dev`, `all`. Supported Python is >= 3.10; CI runs 3.10 – 3.12 on a `[dev]` install and 3.12 again on a core install, which must be green: no test may need data, network or a Bloomberg terminal.

## Conventions

- Test files are named `*_test.py` and live in a `tests/` directory inside the subpackage under test.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ArturSepp/OptimalPortfolios](https://github.com/ArturSepp/OptimalPortfolios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
