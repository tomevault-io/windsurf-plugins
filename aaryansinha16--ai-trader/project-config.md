---
trigger: always_on
description: Intraday NIFTY options paper-trading system. Collects live ticks, trains XGBoost ML models, scores trade signals, and presents everything through a Next.js dashboard backed by a Flask API.
---

# AI Trader — CLAUDE.md

Intraday NIFTY options paper-trading system. Collects live ticks, trains XGBoost ML models, scores trade signals, and presents everything through a Next.js dashboard backed by a Flask API.

---

## Stack

| Layer | Tech |
|---|---|
| Frontend | Next.js 16 + React 19 + Tailwind v4 + Recharts |
| Backend API | Flask (Python 3.13), SSE stream at `/api/stream` |
| Database | TimescaleDB (PostgreSQL 17) — hypertables for tick/candle data |
| ML | XGBoost + LightGBM (scikit-learn pipeline), joblib `.pkl` models |
| Data Feed | TrueData REST + WebSocket (`wss://push.truedata.in:8084`) |
| Runtime | macOS, Python venv at `.venv/`, Node in `dashboard/node_modules/` |

---

## Directory Structure

```
ai-trader/
├── backend/app.py          # Flask API server (port 5050) — the main backend
├── dashboard/               # Next.js frontend (port 3000)
│   └── app/
│       ├── live/            # Live trading page (SSE stream, positions, suggestions)
│       ├── charts/          # Option chain + candle charts
│       ├── backtest/        # Backtest runner + results
│       ├── trades/          # Trade history
│       ├── ai/              # AI chat
│       └── settings/        # Risk profile + config
├── scripts/
│   ├── collect_ticks.py          # LIVE tick collector (runs during market hours)
│   ├── incremental_train.py      # Daily macro/micro model retraining after market close
│   ├── train_outcome_models.py   # Train strategy models on actual trade outcomes (WIN/LOSS)
│   ├── train_rl_on_journeys.py   # Retrain RL exit agent on all saved journey data
│   ├── tick_replay_backtest.py   # Tick-level replay backtest engine
│   ├── fetch_missing_ticks.py    # Backfill single symbol via REST
│   └── backfill_today.py         # Backfill all today's symbols via REST
├── models/
│   ├── train_model.py       # MacroModelTrainer + MicroModelTrainer classes
│   ├── strategy_models.py   # Strategy-specific XGBoost models
│   ├── rl_exit_agent.py     # RLExitAgent (Q-learning, tabular, 8-feature state)
│   ├── predict.py           # Predictor wrapper (load + infer)
│   └── saved/               # .pkl files (macro_model.pkl, micro_model.pkl, rl_exit_agent.pkl)
│       ├── strategy/        # Per-strategy outcome models (bearish_momentum_model.pkl, etc.)
│       └── backups/YYYYMMDD/ # Date-organized model backups before each retrain
├── data/
│   ├── truedata_adapter.py  # TrueData REST + WebSocket client
│   └── tick_collector.py    # TickCollector (buffers 200 ticks → DB flush)
├── features/
│   ├── indicators.py        # compute_all_macro_indicators() — 58 features
│   └── micro_features.py    # compute_micro_features() — 5 features
├── strategy/
│   ├── signal_generator.py  # Generates BUY/SELL signals per strategy
│   ├── trade_scorer.py      # Composite score = 0.5×ML + 0.3×flow + 0.2×tech
│   ├── regime_detector.py   # TRENDING_BULL/BEAR/SIDEWAYS/HIGH_VOL/LOW_VOL
│   └── options_flow_detector.py
├── config/
│   ├── settings.py          # All constants (DB URL, symbols, thresholds)
│   └── risk_profiles.py     # LOW/MEDIUM/HIGH RiskProfile dataclasses
├── database/
│   ├── db.py                # read_sql / write_df / upsert_candles / init_db
│   └── schema.sql           # Full TimescaleDB schema
└── backtest/
    ├── backtest_engine.py
    └── option_resolver.py   # get_nearest_expiry(date) → next Tuesday expiry
```

---

## How to Run

### Prerequisites
```bash
brew services start postgresql@17   # TimescaleDB must be running
cd /Users/aaryansinha/Dev/Projects/ai-trader
source .venv/bin/activate
```

### Backend (Flask API)
```bash
python backend/app.py              # http://localhost:5050
```
Auto-starts `collect_ticks.py` during market hours via `_ensure_collector()`.

### Frontend (Next.js dashboard)
```bash
cd dashboard && npm run dev         # http://localhost:3000
```

### Live Tick Collector (market hours only: 9:15–15:30 IST)
```bash
# Flask auto-starts this. To run manually:
nohup .venv/bin/python scripts/collect_ticks.py >> logs/tick_collector_YYYYMMDD.log 2>&1 &
```
Writes to `/tmp/td_live_prices.json` every 1s (Flask reads this for streaming prices).

### Retrain Models (after market close)
```bash
# Full retrain on ALL historical data (preferred):
python scripts/incremental_train.py

# Incremental only (1–2 new days, uses warm-start):
python scripts/incremental_train.py --days 1

# Full retrain from scratch:
python main.py train
```

### Fill Missing Data
```bash
# Fill today's candles + ticks for NIFTY-I + all ATM options via REST:
python scripts/backfill_today.py

# Fill specific date for a single symbol:
python scripts/fetch_missing_ticks.py --dates 2026-03-25 --symbol NIFTY-I
```

### DB Init / Schema
```bash
python -c "from database.db import init_db; init_db()"
```

---

## Database Schema (TimescaleDB Hypertables)

| Table | Key Columns | Notes |
|---|---|---|
| `tick_data` | `timestamp, symbol, price, volume, oi, bid_price, ask_price` | Hypertable; ~8k ticks/day/symbol |
| `minute_candles` | `timestamp, symbol, open, high, low, close, volume, vwap, oi` | Primary ML training source; upsert by (timestamp, symbol) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aaryansinha16/AI-trader](https://github.com/aaryansinha16/AI-trader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
