---
trigger: always_on
description: A multi-agent AI forex trading system using Claude API as the intelligence layer
---

# CLAUDE.md — Forex Trading System Project Context
# This file is for Claude Code (VS Code) to stay aligned with all decisions,
# architecture, and progress made in the main Claude.ai chat session.
# Last updated: March 28, 2026

---

## PROJECT OVERVIEW

A multi-agent AI forex trading system using Claude API as the intelligence layer
with rule-based Python execution. Currently in 12-month demo phase.

**Owner:** Gazmir Sulcaj (gsulcaj22@gmail.com)
**Start Date:** March 10, 2026
**Demo Period:** 12 months (ends March 10, 2027)
**Live Trading:** Not before March 2027

---

## LOCKED DECISIONS — DO NOT CHANGE

These decisions were made in the main session and are final:

| Decision | Value |
|----------|-------|
| Focus pair | EUR/USD ONLY |
| Broker | OANDA (demo) → IBKR for futures later |
| Starting capital | $5,000 (when live) |
| Demo capital | $100,000 OANDA V20 demo account ✅ ACTIVE |
| OANDA Account ID | 101-001-38764497-001 |
| Agent approach | Option 1 (System Prompt) + Option 2 (RAG) combined |
| Embedding model | all-MiniLM-L6-v2 (local, FREE) |
| Vector DB | ChromaDB (local, FREE) |
| Demo mode flag | Always TRUE until March 2027 |

---

## HARD TRADING RULES — NEVER MODIFY

These rules are embedded in the system prompt and must NEVER be changed:

```
1. Max risk per trade:     1% of account equity
2. Min Risk:Reward ratio:  1:2 (2.0)
3. Daily loss limit:       2% — triggers auto-stop of all trading
4. Confidence threshold:   <65% = NEUTRAL signal always (no trade)
5. News blackout:          30 min before NFP, CPI, FOMC, ECB decisions
6. Kill Zones only:        London (3-4 AM EST) + NY (8-10 AM EST)
7. Demo mode:              TRUE (hardcoded) until month 12
```

---

## SYSTEM ARCHITECTURE

```
main.py
├── RAGPipeline (app/rag/pipeline.py)
│   ├── ChromaDB vector store (chroma_db/)
│   ├── all-MiniLM-L6-v2 embeddings (local)
│   └── 6 collections: books, research, ict, cot, journal, feedback
│
├── ForexAnalystAgent (app/analysis/agent.py)
│   ├── Option 1: System prompt (embedded trading knowledge)
│   ├── Option 2: RAG retrieval (contextual chunks per analysis)
│   └── Claude API → claude-sonnet-4-20250514
│
├── OANDAClient (app/brokers/oanda.py)
│   ├── Live EUR/USD price feed
│   ├── OHLCV candles (4H, 1H, Daily, Weekly)
│   └── Account summary + open positions (+ stop-loss extraction for stop-based open risk)
│
├── MarketDataBuilder (app/brokers/oanda.py)
│   ├── IndicatorCalculator (EMA, RSI, ADX, ATR)
│   ├── MarketStructureAnalyzer (confirmed swing pivots → HH/HL/LH/LL)
│   └── ICT concepts (Order Blocks, FVGs, P/D zones, Liquidity sweeps)
│
├── TradeExecutor (app/execution/trade_executor.py)
│   ├── Order validation + sizing
│   ├── OANDA order placement / monitoring
│   ├── TP1 at configurable fraction of entry→TP2 (currently 60%)
│   ├── TP1 partial close (tp1_close_percent from config)
│   ├── First-hour early momentum exit for stalled trades
│   ├── ATR-based trailing stop after TP1 (tp2_trail from config)
│   └── Trade outcome feedback loop
│
├── TradeJournal (app/execution/trade_journal.py)
│   ├── Open/closed trade persistence
│   ├── Structured feedback fields per closed trade:
│   │   setup_grade (A/B/C/F), entry_timing, ict_post_hoc,
│   │   root_cause, pattern_tags
│   └── Trade signal timeline per trade (one JSON file from entry to close)
│
├── app/backtesting/
│   ├── HistoricalDataLoader (data_loader.py)
│   ├── SignalReplayEngine (signal_replayer.py)
│   ├── replay_confluence.py
│   ├── OutcomeSimulator (outcome_simulator.py)
│   ├── report.py
│   └── HistoricalFundamentalsProvider (historical_fundamentals_provider.py)
│
└── app/fundamentals/fetcher.py
    ├── DXY intraday signal via Yahoo Finance
    ├── COT positioning via CFTC.gov (weekly, delayed)
    ├── Next high-impact event via Forex Factory
    │   refreshed in fixed 6-hour UTC slots
    └── Latest FX headline via Finnhub or NewsAPI when configured
```

---

## FILE STRUCTURE

```
~/forex_trading_system/
├── main.py                    ← Thin CLI entrypoint
├── api_server.py              ← Thin FastAPI entrypoint
├── requirements.txt           ← Dependencies
├── README.md                  ← Setup guide
├── app/
│   ├── analysis/              ← Agent, scheduler, market analysis
│   ├── api/                   ← FastAPI implementation
│   ├── brokers/               ← OANDA market/account data
│   ├── cli/                   ← Runtime CLI implementation
│   ├── core/                  ← Shared config/bootstrap helpers
│   ├── execution/             ← Order execution + monitoring
│   ├── fundamentals/          ← Live fundamentals layer
│   ├── logs/                  ← Signal log helpers
│   └── rag/                   ← RAG pipeline implementation
│
├── documents/
│   ├── books/                 ← Trading books (loaded into RAG)
│   │   ├── DAY TRADING AND SWING TRADING.txt        (160 chunks) ← Kathy Lien
│   │   ├── day-trading-kathy-lien.pdf               (162 chunks) ← duplicate OK
│   │   ├── john_murphy_ocr.txt                      (262 chunks) ← John Murphy (OCR)
│   │   ├── the_forex_trading_course.pdf             (123 chunks) ← Abe Cofnas
│   │   └── trading_in_the_zone_ocr.txt              (157 chunks) ← Mark Douglas (OCR)
│   ├── research/              ← Free BIS/Fed/SSRN papers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gazmi82) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
