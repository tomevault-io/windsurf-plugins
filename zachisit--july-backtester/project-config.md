---
trigger: always_on
description: > **RTK RULE — NON-NEGOTIABLE**: Every terminal command MUST be prefixed with `rtk`. No exceptions. This includes `rtk git`, `rtk grep`, `rtk pytest`, `rtk ls`, `rtk read`, etc. Bare `grep`, `git`, `pytest`, `find` etc. are FORBIDDEN in this project.
---

# CLAUDE.md — july-backtester

> **RTK RULE — NON-NEGOTIABLE**: Every terminal command MUST be prefixed with `rtk`. No exceptions. This includes `rtk git`, `rtk grep`, `rtk pytest`, `rtk ls`, `rtk read`, etc. Bare `grep`, `git`, `pytest`, `find` etc. are FORBIDDEN in this project.

> **GH POSTING RULE — NON-NEGOTIABLE**: Never pass comment/issue/review text as a shell argument. Use `python3 scripts/gh_post.py` for every GitHub comment, review, issue creation, and issue-body edit. `gh issue comment --body "..."` and `gh api -f body=@file` are FORBIDDEN — both have silently destroyed real comments in this project. See [Posting to GitHub](#posting-to-github).

## What This Is
Python backtesting engine for US equities. Tests 20+ technical strategies across single symbols or large portfolios (Nasdaq, S&P 500, etc.) with Monte Carlo robustness scoring.

## Entry Points
- `main.py` — runs all entries in `portfolios`; a single-ticker basket like `"My Symbols": ["AAPL"]` replaces the old single-asset mode
- `main.py --name "run-name"` — optional prefix for report folder and S3 path
- `main.py --verbose` — print two additional tables beneath the default Core Performance table: Extended Metrics (RS(avg/min/last), MaxRcvry, AvgRcvry, Calmar, PF, WinRate, Trades, Expct(R), SQN) and Robustness (OOS P&L, WFA Verdict, RollWFA, Corr, MC, MC Score). Default output shows Core Performance only.

## Versioning

The app version lives in **`version.py`** at the project root (`__version__ = "x.y.z"`). This is the single source of truth — update it whenever a release goes out. It surfaces via `python main.py --version`.

**On every release:** bump `version.py` to the new version string before merging to `main`.

## Key Files
```
version.py                         # Single source of truth for the app version — bump on every release
config.py                          # All settings — edit this before running
main.py                            # Single entry point (--name, --verbose, --dry-run, --init)
helpers/indicators.py              # All strategy signal logic — do not touch
helpers/registry.py                # Strategy registry: register_strategy decorator, load_strategies, get_active_strategies
helpers/simulations.py             # Single-asset trade simulation engine
helpers/portfolio_simulations.py   # Multi-asset portfolio simulation engine
helpers/monte_carlo.py             # Monte Carlo robustness scoring
helpers/summary.py                 # Report generation, S3 upload; dynamic benchmark columns via _build_benchmark_columns(); _get_t1_cols()/_get_t2_cols()/_T3_COLS/_VERBOSE_SHORT_NAMES control tiered table layout; _print_table() handles all bordered terminal output
helpers/wfa.py                     # Walk-Forward Analysis (get_split_date, split_trades, evaluate_wfa)
helpers/wfa_rolling.py             # Rolling multi-fold WFA (get_fold_dates, evaluate_rolling_wfa)
helpers/ml_export.py               # ML trade feature export (export_trade_features)
helpers/sensitivity.py             # Parameter sensitivity sweep (build_param_grid, label_for_params, is_sweep_enabled)
helpers/regime.py                  # VIX regime heatmap (build_regime_heatmap, print_regime_heatmap, classify_vix_regime)
helpers/point_in_time.py           # Point-in-time index universe resolver for portfolios like "pit:nq100" and "pit:sp500"
helpers/rule_based_universe.py     # Survivorship-free universe from the delisted-inclusive Parquet corpus ("rule:top500") — needs no index membership
helpers/init_wizard.py             # First-time setup wizard invoked via python main.py --init
helpers/correlation.py             # Strategy correlation matrix (run_correlation_analysis, compute_avg_correlations)
helpers/caching.py                 # Local Parquet cache (24h TTL)
helpers/aws_utils.py               # S3 upload helper (upload_file_to_s3); reads API key from env or .env via get_secret
helpers/timeframe_utils.py         # Converts '200d' -> bar count for given timeframe
custom_strategies/                 # Plugin directory — drop *.py files here to add strategies
custom_strategies/sma_crossovers.py  # Active strategies: SMA Crossover (20d/50d) and (50d/200d)
services/services.py               # Data provider factory (caching wrapper)
services/polygon_service.py        # Polygon.io REST API
services/norgate_service.py        # Norgate Data local API
services/yahoo_service.py          # Yahoo Finance via yfinance (no API key)
services/csv_service.py            # Local CSV files ({csv_data_dir}/{SYMBOL}.csv)
tickers_to_scan/                   # JSON ticker lists (nasdaq_100.json, sp-500.json, etc.)
scripts/                           # One-off diagnostic and utility scripts (NOT part of the pipeline)
scripts/debug_data.py              # Compares Polygon vs Yahoo SPY data; run with: python scripts/debug_data.py
scripts/build_research_index.py    # Consolidates all output/runs/ summaries + llm_verdict.json into output/research_index.csv

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zachisit/july-backtester](https://github.com/zachisit/july-backtester) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
