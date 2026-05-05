---
trigger: always_on
description: An automated trading bot arena that runs 4 competing bots on Polymarket's BTC 5-minute up/down markets via the Simmer paper trading platform. Bots evolve every 4 hours — the bottom 2 are replaced by mutated copies of the top 2. Each bot has its own Simmer account for independent trading and real performance comparison.
---

# Polymarket Bot Arena — Developer Guide

## What This Is

An automated trading bot arena that runs 4 competing bots on Polymarket's BTC 5-minute up/down markets via the Simmer paper trading platform. Bots evolve every 4 hours — the bottom 2 are replaced by mutated copies of the top 2. Each bot has its own Simmer account for independent trading and real performance comparison.

## Current State (v4 — Feb 15, 2026)

**GitHub:** https://github.com/ThinkEnigmatic/polymarket-bot-arena.git (branch: main)

### Performance (276 real trades)
- **Total P&L: -$52.10** (mostly from old contrarian/overconfident bets)
- momentum-v1: 55.8% WR, -$1.54 (nearly breakeven, BEST bot)
- hybrid-v1: 45.5% WR, -$7.47
- meanrev-v1: 40.4% WR, -$17.72
- sentiment-v1: 40.0% WR, -$25.36

### Key Data Insights (use these for future iterations)
- **Market price is the strongest signal** — when YES is priced >65c, YES wins ~100% of the time
- **Contrarian/mean-reversion strategies lose money** in 5-min markets
- **Confidence 0.30-0.50 is the sweet spot** — 67.9% WR, +$48 total
- **Confidence >0.50 LOSES money** — 48.6% WR but large bet sizes = big losses
- **NO bets have 44.9% WR vs YES at 49.2%** — slight YES bias is profitable
- **Buying cheap YES (<40c) against market consensus = 0-10% WR** (catastrophic)

### What's Running
- **Arena process:** launchd service `com.polymarket.botarena` — auto-restarts on crash
- **Dashboard:** launchd service `com.polymarket.dashboard` — FastAPI on port 8050
- **Remote access:** localtunnel (not persistent, needs manual restart: `npx localtunnel --port 8050`)
- **Price feed:** Binance WebSocket for BTC/USDT 1-min candles

### Simmer API Keys (4 accounts, slot-based)
Stored at `~/.config/simmer/bot_keys.json` — keys mapped to slot_0 through slot_3. When evolution kills a bot, the replacement inherits the dead bot's slot (and API key). Default key at `~/.config/simmer/simmer_api_key.json`.

## Architecture

### Signal Hierarchy (make_decision in base_bot.py)
```
combined = (
    market_price_edge * 0.50    # Strongest: follow the market price
    + btc_momentum * 0.20       # BTC price movement direction
    + strategy_signal * 0.15    # Per-bot strategy differentiation
    + learning_bias * variable  # Grows from 10% to 60% weight with data
)
```

### Safeguards
- **Market consensus guard:** Never bet against prices >65c or <35c
- **Bet sizing cap:** Confidence capped at 0.45 for sizing (prevents overconfident large bets)
- **Stale trade expiry:** Trades pending >1h auto-expire (5-min markets resolve in ~10min)
- **Daily loss limits:** Uncapped for paper trading (was $10/bot, $25 total)
- **Dedup:** Loads recent (bot, market) pairs from DB to prevent duplicates across restarts

### Per-Strategy Differentiation
| Strategy | Aggression | Prior | Min Confidence |
|----------|-----------|-------|----------------|
| momentum | 1.2 (follows price strongly) | 0.52 (slight YES) | 0.01 (trades almost everything) |
| mean_reversion | 0.95 (nearly follows, was 0.6) | 0.48 (slight NO) | 0.06 |
| sentiment | 1.0 (neutral) | 0.50 | 0.03 |
| hybrid | 1.0 (neutral, was 0.9) | 0.50 | 0.05 |

### Learning System
- Features extracted at TRADE TIME (not resolution time — this was a critical bug fix)
- Stored in `trade_features` column in trades table
- Learning records win/loss by feature bucket (price level + momentum)
- Weight ramps from 10% to 60% as bot accumulates resolved trades
- Learning data in `bot_learning` table

## Key Files

```
arena.py              # Main loop: discover markets, run bots, resolve trades, evolve
bots/base_bot.py      # BaseBot with make_decision() signal hierarchy + execute()
bots/bot_momentum.py  # MomentumBot (follows trends)
bots/bot_mean_rev.py  # MeanRevBot (was contrarian, now nearly neutral)
bots/bot_sentiment.py # SentimentBot
bots/bot_hybrid.py    # HybridBot
config.py             # All config: paths, limits, evolution interval, API URLs
db.py                 # SQLite: trades, bot_configs, evolution_events, bot_learning
learning.py           # Feature extraction, bias calculation, outcome recording
signals/price_feed.py # Binance WS for BTC candles (staleness detection)
signals/sentiment.py  # Sentiment signals
signals/orderflow.py  # Order flow signals
dashboard/server.py   # FastAPI dashboard backend
dashboard/index.html  # Dashboard frontend
copytrading/          # Wallet tracking + copy trading (not actively used)
```

### launchd Services
```
~/Library/LaunchAgents/com.polymarket.botarena.plist  → arena.py
~/Library/LaunchAgents/com.polymarket.dashboard.plist → dashboard/server.py
```

### Database
SQLite at `trading_bot/data/arena.db` — tables: trades, bot_configs, evolution_events, daily_stats, bot_learning, copytrading_wallets, copytrading_trades

## Bug History (avoid re-introducing)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ThinkEnigmatic/polymarket-bot-arena](https://github.com/ThinkEnigmatic/polymarket-bot-arena) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
