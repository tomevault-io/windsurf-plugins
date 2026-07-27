---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**ETF Rotation Strategy Research Platform** — A-share + QDII markets. Three-tier engine (WFO → VEC → BT) screens factor combinations, backtests them, and produces sealed production strategies. Current production: **v8.0** with FREQ=5, Exp4 hysteresis enabled, ICIR-weighted scoring, and stateful signal generation.

## Environment & Commands

**Package manager: UV only.** Never use `pip install`, `python -m venv`, or bare `python <script>`.

```bash
uv sync --dev                    # Install all dependencies

# Makefile shortcuts
make wfo                         # WFO screening (~2min)
make vec                         # VEC backtesting (~5min)
make bt                          # BT audit (~30-60min)
make pipeline                    # Full WFO → VEC → BT pipeline (~76s)
make all                         # wfo + vec + bt
make format                      # black + isort
make lint                        # ruff + mypy
make check                       # pre-commit --all-files
make test                        # pytest -v
make test-cov                    # pytest with coverage
make clean-numba                 # Clear Numba JIT cache (required after @njit signature changes)
make signal                      # Daily trading signal (shortcut)
make validate                    # Final triple validation
make update-data                 # Update all market data (incremental)
make research                    # Full research pipeline: mining → WFO → VEC → BT → validation
make pipeline-fast               # Pipeline with 16 BT workers, 16 VEC threads

# Direct commands
uv run python src/etf_strategy/run_combo_wfo.py           # WFO screening
uv run python scripts/batch_vec_backtest.py                # VEC backtesting
uv run python scripts/batch_bt_backtest.py                 # BT ground truth audit
uv run python scripts/precompute_non_ohlcv_factors.py      # Precompute non-OHLCV factors (run before pipeline)
uv run python scripts/run_full_pipeline.py                 # Full pipeline (WFO → VEC → BT)
uv run python scripts/generate_today_signal.py             # Daily trading signal (stateful)
uv run python scripts/update_daily_from_qmt_bridge.py --all  # Data update from QMT

# Testing
uv run pytest tests/ -v                                    # All tests (210 cases)
uv run pytest tests/test_frozen_params.py -v               # Single file
uv run pytest -k "test_shift" -v                           # Single test by name
```

## Three-Tier Engine Architecture

```
WFO (screening)  →  VEC (precision)  →  BT (ground truth)
~2 min               ~5 min              ~30-60 min
IC gate + scoring    Numba JIT kernel    Backtrader event-driven
```

- **WFO** (`src/etf_strategy/run_combo_wfo.py`): Screens 12,597 factor combinations (sizes 2-7) using rolling IC as gate (≥0.05 or ≥55% positive rate), then ranks by composite score: Return(40%) + Sharpe(30%) + MaxDD(30%). IC alone has only 0.0319 correlation with actual returns — never rank by IC.
- **VEC** (`scripts/batch_vec_backtest.py`): Numba-accelerated vectorized backtest for top candidates. Fast but uses float shares.
- **BT** (`scripts/batch_bt_backtest.py`): Backtrader event-driven simulation with integer lots and capital constraints. Production ground truth.
- **Validation**: Rolling OOS (≥60% positive windows) + Holdout (return > 0) via `final_triple_validation.py`.

## Production Parameters (v8.0)

Enforced by `src/etf_strategy/core/frozen_params.py` — validated at WFO/VEC/BT entry points. Override with `FROZEN_PARAMS_MODE=warn` for A/B testing only.

| Parameter | Value | Notes |
|-----------|-------|-------|
| `FREQ` | 5 | Rebalance every 5 trading days |
| `POS_SIZE` | 2 | Hold 2 ETFs |
| `COMMISSION` | 0.0002 (2bp) | |
| `LOOKBACK` | 252 | 1 year |
| `delta_rank` | 0.10 | Hysteresis: min rank01 gap for swap |
| `min_hold_days` | 9 | Hysteresis: min holding period |
| ETF pool | 49 (41 A-share + 8 QDII) | |
| Universe mode | `A_SHARE_ONLY` | QDII hard-blocked from live trading |

**Version registry**: v3.4/v4.0/v4.1 preserved with freq=3 and hysteresis disabled for rollback. `CURRENT_VERSION = "v8.0"`.

**v8.0 strategies**:
- Champion `composite_1`: `ADX_14D + BREAKOUT_20D + MARGIN_BUY_RATIO + PRICE_POSITION_120D + SHARE_CHG_5D` (5F)
- Fallback `core_4f`: `MARGIN_CHG_10D + PRICE_POSITION_120D + SHARE_CHG_20D + SLOPE_20D` (4F)

**8 QDII ETFs** (monitored but not traded in A_SHARE_ONLY mode):
159920 (恒生ETF), 513050 (中概互联网ETF), 513100 (纳指ETF), 513130 (恒生科技ETF), 513180 (恒生科技指数ETF), 513400 (道琼斯ETF), 513500 (标普500ETF), 513520 (日经ETF)

## Hysteresis State Machine

`src/etf_strategy/core/hysteresis.py` — `@njit` kernel shared by WFO/VEC/BT.

Rules:
1. Max 1 swap per rebalance (forced)
2. Swap only if rank_gap ≥ delta_rank (0.10)
3. Swap only if held_days ≥ min_hold_days (9)

**Signal generator state persistence**: `data/live/signal_state.json` stores per-strategy portfolios and hold_days between daily runs. Schema includes `version`, `freq`, `universe_mode` for env-mismatch detection (auto cold-start if validation fails). State only updates on rebalance days.

## Required Shared Utilities


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zhangsensen/etf-rotation-strategy](https://github.com/zhangsensen/etf-rotation-strategy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
