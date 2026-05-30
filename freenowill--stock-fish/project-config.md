---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Start server (port 8000)
python app.py

# One-click deploy (Docker or local)
bash run.sh              # Docker: pulls zhuhai123/stockfish-* images, starts StockFish + MiroFish
bash run.sh --local      # Local Python (no Docker, port 8000)
bash run.sh --no-mirofish # Docker: StockFish only, skip MiroFish

# API: multi-factor analysis (~2 min)
curl -X POST http://localhost:8000/api/analyze \
  -H 'Content-Type: application/json' \
  -d '{"symbol":"600519","cost_price":150}'

# API: prediction pipeline (async, ~15 min, SSE progress stream)
curl -X POST http://localhost:8000/api/predict \
  -H 'Content-Type: application/json' \
  -d '{"symbol":"600519","scenario":"base"}'

# SSE progress stream
curl -N http://localhost:8000/api/predict/<task_id>/stream

# System config
curl http://localhost:8000/api/config

# Install dependencies
pip install -r requirements.txt
```

No test framework exists (`tests/__init__.py` is empty). No linting/formatting config.

## Architecture

**5-step analysis pipeline** (`POST /api/analyze`):

1. **Data Collection** (`analysis/agent.py:StockAnalysisAgent.analyze`) — fetches quote, technical indicators, financials, news, guba posts via `AStockProvider` (auto-selects backend). Supports A-shares, BSE stocks (`.BJ` suffix auto-converted for Tushare), HK/US stocks.
2. **Sentiment** (`market_data/sentiment_collector.py`) — HuggingFace multilingual model → 5-class sentiment, with keyword-rule fallback. Reuses BettaFish's model if available.
3. **Valuation + Signal Generation** (`analysis/scoring.py:ScoringEngine`) — PE percentile → valuation level (很低~很高), suggested buy price (PE mean-reversion + Bollinger lower support). -5~+5 composite score: technical (RSI/MACD/MA/Bollinger/volume/momentum) + fundamental (PE/ROE/growth/dividend) + sentiment (news/guba). Adaptive weights based on market regime.
4. **LLM Prediction** (`analysis/nodes/prediction_node.py`) — 3 parallel agents (tech/fundamental/sentiment) each analyze only their domain data, then a Moderator reads all views and produces final prediction with multi-cycle outlook (short/mid/long term), suggested action, stop-loss/take-profit.
5. **Frontend rendering** — `static/index.html`: dark-theme single-page UI with 6 cards, technical/fundamental detail grids, sentiment bars, multi-agent debate panel, score breakdown, news/guba summaries.

**Simulation bridge** (`POST /api/predict`, background thread):
- `analysis/agent.py` → `simulation_bridge/seed_builder.py` → `simulation_bridge/orchestrator.py` → MiroFish HTTP API
- Seed document embeds **7 agent roles** (Buffett/Munger/Valuation/Sentiment/Fundamental/Technical/RiskManager) + explicit entity-relationship statements for Zep GraphRAG extraction
- Pipeline: ontology generation → graph build → simulation create → agent profile prep → OASIS run → report generation
- Each stage has polling with configurable timeouts; falls back to standalone mode on any MiroFish failure
- SSE progress stream with real-time log messages (EventSource in frontend, polling fallback)

## Data Backends (`market_data/a_stock_provider.py`)

`AStockProvider` auto-selects (5 backends): `advanced` → `tushare` → `akshare` → `baostock` → `mock`. Config via `STOCK_BACKEND` env var (default: `mock`).

- **MockBackend** — Random data, zero network, for dev/demo
- **AkShareBackend** — EastMoney data via akshare (needs mainland China network)
- **BaoStockBackend** — Free, no token, academic-grade fallback
- **TushareBackend** (`tushare_provider.py`) — Tushare Pro via vendored `sxsc_tushare` SDK (山西证券 proxy). Supports `.BJ` suffix for BSE stocks.
- **AdvancedBackend** (`provider_adapter.py`) — DataFetcherManager wrapping 11 fetchers (efinance/akshare/tushare/pytdx/baostock/yfinance/finnhub/alphavantage/longbridge) with circuit-breaker failover, 7 search engines (Tavily/Bocha/Brave/SerpAPI/Anspire/MiniMax/SearXNG), social sentiment (Reddit/X/Polymarket, US stocks only), fundamental pipeline (growth/earnings/institutional/capital flow/dragon-tiger boards).

All backends implement `BaseStockBackend`: `get_quote`, `get_historical`, `get_financials`, `get_news`, `get_guba`, `get_historical_pe`.

Real-time quote priority chain (configurable via `REALTIME_SOURCE_PRIORITY`): Tencent → Sina → Eastmoney → Tushare.

## News Sources (`market_data/news_sources.py`)

Plugin architecture: each source extends `BaseNewsSource` or `BaseGubaSource`, registered in `NEWS_SOURCES`/`GUBA_SOURCES` lists. Current: SinaNews, NewsNow (cls+xueqiu+wallstreetcn aggregate), YahooFinance, XueqiuPopularity, CLSNews (disabled), EastMoneyGuba.

## Frontend (`static/index.html`)

Single-file dark-theme SPA (660 lines, no build step). Features:
- Stock symbol + cost price input, optional "智能推演" checkbox (toggles between `/api/analyze` and `/api/predict`)
- SSE real-time log streaming during simulation, polling fallback on SSE error

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [freenowill/stock-fish](https://github.com/freenowill/stock-fish) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-30 -->
