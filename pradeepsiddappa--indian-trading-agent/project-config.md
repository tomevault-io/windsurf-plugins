---
trigger: always_on
description: AI-powered multi-agent trading decision system for Indian markets (NSE/BSE).
---

# Indian Market Trading Agent

AI-powered multi-agent trading decision system for Indian markets (NSE/BSE).
Built on [TradingAgents](https://github.com/TauricResearch/TradingAgents) framework with LangGraph.

## Architecture

```
frontend/ (Next.js 16 + Tailwind + shadcn/ui + Open Sans)  :3000
    |
backend/ (FastAPI + WebSocket)                             :8000
    |
tradingagents/ (LangGraph multi-agent pipeline)
    |
yfinance + RSS feeds (NSE/BSE data + news)
```

## Quick Start

```bash
# 1. Install Python deps
python3 -m venv venv && source venv/bin/activate
pip install -e .
pip install fastapi uvicorn websockets aiosqlite numpy feedparser

# 2. Configure API keys (2 options)
# Option A: via .env
echo 'ANTHROPIC_API_KEY=your_key' > .env
# Option B: via Settings page in the UI (stored in SQLite, takes priority)

# 3. Start backend
uvicorn backend.app:app --reload --port 8000

# 4. Start frontend (separate terminal)
cd frontend && npm install && npm run dev

# 5. Open http://localhost:3000
```

## Information Architecture

The UI is organized by daily workflow (not by technical feature):

```
🏠 Today              — Daily starting page with auto-loaded top picks + workflow guide

DISCOVER
  ✨ Top Picks         — AI-free unified recommendation engine (combines all signals)
  📡 Market Scan       — Gap / Volume / Breakout scanner
  🎯 Strategies        — S/R, Pivot Points, Cyclical Patterns (seasonality, sector rotation)
  📰 News Feed         — Aggregated Indian market news (RSS + yfinance, customizable)

ANALYZE
  🔍 Deep Analysis     — AI-powered 10-agent pipeline (paid ~Rs.15-60)
  📊 Charts            — Candlestick charts with volume

VALIDATE
  🏆 Performance       — Historical win rate of each strategy (FREE)
  🧪 Simulation        — Paper trading + historical recommender backtest (FREE)
  🧠 Learning Insights — Pattern analysis of YOUR past trades (FREE, no ML)
  📈 Signal Performance — Per-signal win rate + auto-tune recommender (FREE)
  🎯 Verdict Calibration — Is the daily verdict actually predictive? (FREE)
  ⚖️ Confidence Calibration — Brier score: are stated probabilities honest? (FREE)
  👁️ Shadow Trades       — Counterfactual auto-tracking of skipped picks (FREE)
  🧠 Memory Admin        — Inspect + prune agent BM25 memories (FREE)
  🔬 AI Backtest       — Run AI pipeline on past dates (paid)
  📋 My Trades         — History with P&L tracking + "Teach the agent" reflection

⚙️ Settings           — API keys (UI), LLM provider switcher, model selection, cost guide
```

## Features

### FREE Features (no API cost)
- **Top Picks / Recommendations** — Combines 10+ signals (gap, volume, breakout, S/R, RSI, cyclical, trend) into ranked trade ideas with success probability %
- **Market Scanner** — Finds stocks with gap up/down, volume spikes, 20-day breakouts
- **Support/Resistance** — S1-S3 / R1-R3 levels from historical price action + daily Pivot Points
- **Cyclical Patterns** — Monthly seasonality, day-of-week patterns, sector rotation (9 sectors)
- **Seasonal Backtest** — "Buy in January, sell in March" strategy testing with pure price math
- **Performance Tracker** — Historical win rate of Gap/Volume/Breakout/S/R-Bounce strategies over 30/60/90 days
- **News Feed** — Aggregated from 7 Indian RSS sources (MoneyControl, ET, LiveMint, Business Standard, NDTV Profit) + yfinance search queries. Fully customizable (add/remove/edit sources).
- **Charts** — Interactive candlestick with volume (TradingView lightweight-charts)
- **Watchlist** — Persistent across sessions (SQLite)
- **Workflow Guide** — 3-step visual guide on Dashboard

### Paid Features (AI API cost)
- **Deep Analysis** — Full 10-agent pipeline with customization:
  - **Analyst selection** — pick Market/Social/News/Fundamentals (min 1)
  - **Research depth** — Shallow (1 round) / Medium (2) / Deep (3)
  - **Output language** — English / Hindi
  - **Live cost estimate** before you run
  - **Stats tracking** — exact tokens, LLM calls, cost in Rs./USD after completion
  - Per-model breakdown of token usage
  - Cost: ~Rs.8-70 depending on config (Haiku+Sonnet mix)
- **AI Backtest** — Runs Deep Analysis on past dates with P&L tracking + optional learning

### Agent Learning System
- **Memory persistence** — All 5 agent memories (Bull, Bear, Trader, Judge, Portfolio Manager) auto-save to `~/.tradingagents/memory/*.json`
- **Auto-load** on every backend startup
- **Reflect & Remember from History** — After a trade closes, click "Log P&L" on the trade:
  - Enter entry/exit prices → calculates P&L
  - Optional "Teach the agent" checkbox → runs reflection
  - 5 agents reflect on what went right/wrong
  - Lessons added to memory, used in future analyses
  - Shows "Agent Memory: N lessons learned" badge on History page
- **Important**: The LLM itself is NOT fine-tuned — memory is keyword-retrieved via BM25 and injected into future prompts as context

### Paper Trading Simulation (FREE)
- **Paper Trades** — Click "Track" on any Top Pick or Recommendation → opens virtual position at current market price
- **Multi-horizon tracking** — auto-fetches actual prices at 1/3/5/10 trading days later
- **Captures full context** — source (recommendation/scanner/manual), strategy name, triggered signals, confidence, score

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pradeepsiddappa/indian-trading-agent](https://github.com/pradeepsiddappa/indian-trading-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
