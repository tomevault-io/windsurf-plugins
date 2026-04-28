---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RenQuant is a personal quantitative trading workstation for Apple Silicon. It implements a "Glass-box" pipeline: data ingestion → ML signal generation → backtesting (LEAN) → live trading (Alpaca/IBKR). All components are statistically interpretable and strictly decoupled.

## Environment Setup

Single conda environment (Miniconda, Apple Silicon arm64):

```bash
conda create -n renquant python=3.10
conda activate renquant
pip install pandas numpy matplotlib seaborn yfinance scikit-learn xgboost jupyterlab pyarrow
pip install "openbb[all]" openbb-cli backtesting scipy
pip install lean alpaca-py
lean login
```

Docker must be allocated 16GB+ memory for LEAN engine.

## Workflow: Four Modes

**Research mode** (fast iteration, no Docker): Run notebooks to train and export models.

**Validation mode** (final backtest): Run LEAN on exported models.

```bash
# Single symbol export
python scripts/export_lean_data.py --symbol NVDA

# Batch export: all watchlist symbols for a strategy (recommended before first backtest)
python scripts/export_lean_watchlist.py --strategy renquant_102
python scripts/export_lean_watchlist.py --strategy renquant_102 --force  # re-export all
python scripts/export_lean_watchlist.py --strategy renquant_102 --symbols CRM SHOP  # specific symbols

cd backtesting/renquant_101
lean backtest .
```

**IMPORTANT**: LEAN uses its own data files at `backtesting/data/equity/usa/daily/`, NOT the yfinance parquet cache at `data/ohlcv/`. After training models in a notebook, always run `export_lean_watchlist.py` before backtesting to ensure LEAN has data for all watchlist symbols. Missing data causes silent failures (History() returns empty, no trades execute).

To backtest and render charts in one step (with notifications):

```bash
python scripts/backtest_and_analyze.py --strategy renquant_101
python scripts/backtest_and_analyze.py --strategy renquant_101 --ntfy other  # custom ntfy topic
python scripts/backtest_and_analyze.py --strategy renquant_101 --silent      # no notifications
```

Notifications (on by default, `--silent` to disable): macOS banner via `terminal-notifier` (`brew install terminal-notifier`) and iPhone push via [ntfy.sh](https://ntfy.sh) (default topic: `renquant`, override with `--ntfy <topic>`).

**Analysis mode**: Run `python scripts/analyze_backtest.py --strategy renquant_101` to visualize LEAN results, including decision telemetry for score/threshold inspection.

**Live mode**: Run the live trader with paper, Alpaca, or IBKR broker.

```bash
python -m live.runner --strategy renquant_101 --broker paper --once
python -m live.runner --strategy renquant_102 --broker alpaca-paper --once  # multi-stock
python -m live.runner --strategy renquant_102 --broker alpaca --once  # real money
```

**Scheduled mode**: Three daily automation runs via macOS launchd (all NYSE-holiday-aware):

| Run | Time (PT) | Time (ET) | Script | What it does |
|-----|-----------|-----------|--------|--------------|
| Market open | 6:32 AM | 9:32 AM | `live_only_103.sh --sell-only` | Exit stop-loss / gap-down positions early using today's opening price |
| Pre-close | 12:44 PM | 3:44 PM | `live_only_103.sh --sell-only` | Exit intraday stop breaches before close using near-final daily price |
| After close | 1:55 PM | 4:55 PM | `daily_103.sh` | Full run: retrain models → export LEAN data → buy + sell signals |

```bash
# Manual runs
bash scripts/daily_103.sh              # full run (retrain + trade)
bash scripts/live_only_103.sh          # intraday sell check (no retrain)
python -m live.runner --strategy renquant_103 --broker alpaca --once --sell-only

# LaunchAgents (all loaded)
# ~/Library/LaunchAgents/com.renquant.open103.plist
# ~/Library/LaunchAgents/com.renquant.preclose103.plist
# ~/Library/LaunchAgents/com.renquant.daily103.plist
# Logs: logs/live_103/{date}-open.log, {date}-preclose.log
#       logs/daily_103/{date}.log
# NYSE calendar guard: all three scripts skip US market holidays automatically
```

Alpaca credentials are stored in `.env` (gitignored): `ALPACA_API_KEY` and `ALPACA_SECRET_KEY`. Notifications (macOS + iPhone/ntfy) are sent on success or failure.

## Shared Library: `common/`

Import as `import common`. **Do not import `common/` from inside `backtesting/` — LEAN Docker cannot access it.**

| Module | Key exports |
|--------|-------------|
| `common/config.py` | `load_strategy_config`, `build_model_path` |
| `common/data/` | `fetch_ohlcv` (with Parquet cache), `DataSource` ABC, `LocalStore` |
| `common/indicators/` | `compute_indicators`, `add_indicators` (compat), `list_indicators`, `@register`; regime detection: `compute_hurst`, `rolling_hurst`, `compute_cusum`, `rolling_cusum`, `build_gmm_features`, `RegimeGMM` |
| `common/models/` | `BaseModel`, `ManualModel`, `ClassificationModel`, `QLearningModel`, `FQIModel`, `OptimizationModel`, `XGBoostModel`, `create_model`; all models implement `predict_score_bulk(df)` returning continuous float scores for live ranking |
| `common/models/learners/` | `RTLearner`, `BagLearner`, `TabularQLearner` |
| `common/strategy.py` | `StrategyConfig`, `Strategy` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hallovorld) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-15 -->
