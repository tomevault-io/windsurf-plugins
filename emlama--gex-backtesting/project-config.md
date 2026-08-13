---
trigger: always_on
description: SPX 0DTE Gamma Exposure (GEX) backtesting toolkit for analyzing whether gamma-related metrics
---

# GEX Backtesting Toolkit

## Overview

SPX 0DTE Gamma Exposure (GEX) backtesting toolkit for analyzing whether gamma-related metrics
predict late-day PUT price explosions. Built on 513 days of enriched options trade data
(2024-01-02 through 2026-02-19) from Polygon.io flat files with quote-matched trade side classification.

### Research Question

> Can dealer gamma concentration and convexity metrics (GCI, PGR, GDW, CAR, Charm, Vomma, Zomma)
> measured during the 2:00-3:45 PM ET window predict 0DTE PUT option price explosions
> (>100% gain within 15-60 minutes)?

## Directory Structure

```
gex-backtesting/
├── CLAUDE.md              # This file
├── README.md              # GitHub landing page
├── pyproject.toml         # uv/pip project config (Python >=3.10)
├── download_data.sh       # Download 513-day dataset (2.3 GB) from data server
├── setup_data.sh          # Symlink parquets from Hermes repo instead
├── .gitignore
├── data/                  # Trade parquets (NOT in git)
│   └── trades_YYYY-MM-DD.parquet
├── src/                   # Analysis library (see src/CLAUDE.md)
│   ├── __init__.py
│   ├── config.py          # All thresholds and parameters (dataclasses)
│   ├── data_loader.py     # DataLoader (backtest) + GEXDataLoader (charts)
│   ├── processor.py       # BacktestRunner orchestration
│   ├── metrics.py         # GCI, PGR, GDW, CAR metric calculations
│   ├── gex_calculator.py  # Side-weighted GEX (traditional + directional)
│   ├── greeks.py          # Higher-order Greeks class (gamma, vomma, zomma, charm)
│   ├── black_scholes.py   # Vectorized BS functions (IV, delta, gamma) ~100x faster
│   ├── put_tracker.py     # PUT selection + return measurement
│   ├── statistics.py      # Spearman, Fisher's exact, FDR, permutation tests
│   └── visualization.py   # matplotlib/seaborn chart generation
├── notebooks/
│   ├── 0dte_gex_charts.ipynb          # Daily GEX visualization (single-date explorer)
│   ├── 01_gci_spike_analysis.ipynb    # GCI concentration study (Fisher's exact)
│   └── 02_multi_metric_backtest.ipynb # Multi-metric pre-registered hypothesis test
└── results/               # Generated output (not in git)
```

## Quick Start

```bash
# 1. Install with uv (recommended)
uv venv && source .venv/bin/activate
uv pip install -e ".[jupyter]"

# 2. Download data (513 days, 2.3 GB)
./download_data.sh

# Or symlink from Hermes repo
./setup_data.sh /path/to/parquets

# 3. Run notebooks
jupyter lab notebooks/
```

Start with `0dte_gex_charts.ipynb` -- set `TRADE_DATE` to any date and run all cells.

## Architecture: Two Pipelines

### Pipeline 1: GEX Chart Generation (single-date exploration)

Used by `0dte_gex_charts.ipynb`. Produces daily GEX visualizations: Net Drift, Net Flow,
GEX by strike, DEX, Vomma/Zomma surfaces, CAR, Volatility Skew, Trade Distribution.

```
AnalysisConfig(trade_date="2024-06-14")
    -> GEXDataLoader.load()        # Enriches raw trades with opt_type, strike, trade_dir, tte_years
    -> calculate_gex(df, config)   # Side-weighted + traditional GEX via black_scholes.py
    -> GEXResult.by_strike         # DataFrame for charting
```

Key classes: `AnalysisConfig`, `GEXDataLoader`, `calculate_gex()`, `GEXResult`

### Pipeline 2: Multi-Metric Backtest (cross-date statistical analysis)

Used by `02_multi_metric_backtest.ipynb`. Runs across all 513 days to test whether
gamma metrics predict PUT explosions.

```
Config()
    -> BacktestRunner.run()                    # Loop over all dates
        -> DayProcessor.process(date)          # Per-day orchestration
            -> DataLoader.load_and_prepare()   # Load + filter late-day + create intervals
            -> MetricCalculator.calculate()    # GCI, PGR, GDW, CAR per interval (uses greeks.py)
            -> PutTracker.calculate_returns()  # Track PUT prices at 15/30/45/60 min horizons
    -> BacktestRunner.run_univariate_analysis()   # Spearman + lift per metric
    -> BacktestRunner.run_composite_analysis()    # Combined signals
    -> BacktestRunner.run_control_experiments()   # Placebo, time-shifted, permutation
```

Key classes: `Config`, `BacktestRunner`, `DayProcessor`, `DataLoader`, `MetricCalculator`,
`PutTracker`, `StatisticalAnalyzer`

## Key Domain Concepts

| Metric | Full Name | What It Measures |
|--------|-----------|------------------|
| **GEX** | Gamma Exposure | Dealer gamma position by strike. Positive = stabilizing, negative = destabilizing |
| **Side-Weighted GEX** | -- | GEX using actual trade direction (buy/sell) instead of assuming all customer buys |
| **GCI** | Gamma Concentration Index | Herfindahl index of gamma by strike. High = gamma at few strikes = fragile |
| **PGR** | Protective Gamma Ratio | Fraction of gamma within +/-$20 of spot. Low = gamma far from spot = less protection |
| **GDW** | Gamma Distance Weighted | Exponentially-weighted gamma by distance from spot (decay=20) |
| **CAR** | Convexity Acceleration Risk | Combines zomma + vomma with time amplifier. Measures gamma acceleration risk |
| **Charm** | Delta Decay | Rate of delta change over time. Extreme for 0DTE near expiry |
| **Vomma** | Vol-of-Vol Sensitivity | How vega changes with volatility. High = prices explode on vol spikes |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [emlama/gex-backtesting](https://github.com/emlama/gex-backtesting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
