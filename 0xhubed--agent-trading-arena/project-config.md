---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Agent Arena is an AI-powered cryptocurrency futures trading simulation where LLM-based agents compete using real Kraken Futures market data. Agents make trading decisions on crypto futures (BTC, ETH, SOL, XRP, DOGE) and compete on a profitability leaderboard.

## Commands

```bash
# Install in development mode (core + dev + api)
pip install -e ".[dev,api]"

# Optional extras
pip install -e ".[learning]"   # Postgres + pgvector + OpenAI embeddings

# Run single-tick demo (CLI)
agent-arena demo

# Run competition from config file (CLI)
agent-arena run configs/production.yaml
agent-arena run configs/local_inference.yaml  # Local/Together AI models

# Create new config template
agent-arena init

# Run API server with dashboard
uvicorn agent_arena.api.app:app --reload --port 8000

# Run React frontend (in separate terminal)
cd frontend && npm install && npm run dev

# Lint
ruff check agent_arena/

# Run tests
pytest
pytest tests/test_specific.py -k "test_name"  # single test

# Historical data management
agent-arena data-status                           # Show available data
agent-arena fetch-data -S 2025-01-01 -s PF_XBTUSD # Fetch from Kraken Futures

# Contagion analysis
agent-arena contagion analyze           # System health / echo chamber risk
agent-arena contagion history           # Contagion history

# Scenario management
agent-arena scenario curate             # Curate Kraken Futures data as replayable scenarios
agent-arena scenario list               # List saved scenarios
agent-arena scenario verify             # Verify scenario integrity

# Codegen (journal-driven code fixes)
agent-arena codegen --dry-run                    # Show findings only
agent-arena codegen --max-changes 1 --no-pr      # One fix, no PR
agent-arena codegen                              # Full: branch + edits + PR
agent-arena codegen --lookback-days 10           # Scan more journal entries
```

## Architecture

### Core Design Principle: Stable Core, Flexible Edges

- **Stable (rarely changes):** Competition runner, P&L calculation, Decision interface, storage protocol
- **Flexible (change often):** Agent implementations, data providers, LLM integrations, prompting strategies

### Module Map

```
agent_arena/
├── core/               # Stable core abstractions
│   ├── agent.py        # BaseAgent interface
│   ├── arena.py        # TradingArena (futures simulation)
│   ├── config.py       # CompetitionConfig
│   ├── config_parser.py# Parse fees, constraints, candles from YAML
│   ├── context_builder.py # Enriched context for RAG/storage
│   ├── embeddings.py   # Embedding service for similarity search
│   ├── indicators.py   # RSI, SMA, MACD, ADX, ATR, Bollinger Bands
│   ├── loader.py       # Dynamic agent loading (allowlisted prefixes)
│   ├── models.py       # Decision, PortfolioAnalytics
│   ├── outcomes.py     # Decision outcome scoring
│   ├── regime.py       # Market regime classification
│   └── runner.py       # CompetitionRunner (tick loop)
├── agents/             # All agent implementations (see Agent Types)
├── agentic/            # LangGraph ReAct framework + tools + memory
├── analysis/           # Post-hoc analysis tools
│   ├── bias_scan.py    # Behavioral bias calculators (DEACTIVATED, code preserved)
│   ├── bias_models.py  # BiasProfile, BiasScore dataclasses (DEACTIVATED)
│   ├── contagion.py    # System health / echo chamber detection
│   └── statistics.py   # Statistical utilities
├── api/                # FastAPI REST + WebSocket
│   ├── app.py          # Main app, startup, competition lifecycle
│   ├── routes.py       # Core endpoints (leaderboard, agents, market)
│   ├── routes_backtest.py  # Backtest management
│   ├── routes_evolution.py # Evolution run management
│   ├── routes_forum.py     # Forum messages + witness
│   ├── routes_journal.py   # Observer journal entries
│   ├── routes_lab.py       # Lab (bias + contagion)
│   └── websocket.py    # Real-time event streaming
├── backtest/           # Historical backtesting (DEACTIVATED, code preserved)
│   ├── runner.py       # BacktestRunner (parallel agents, cost estimation)
│   └── results.py      # BacktestResult, AgentResult, TradeRecord
├── data/               # Data management
│   └── fetch_historical.py # Fetch + store Kraken Futures historical data
├── evolution/          # Genetic algorithm optimization (DEACTIVATED, code preserved)
│   ├── engine.py       # EvolutionEngine
│   ├── fitness.py      # FitnessEvaluator (multi-objective scoring)
│   ├── genome.py       # AgentGenome
│   ├── islands.py      # Island model (parallel sub-populations)
│   ├── llm_operators.py# LLM-assisted crossover and mutation
│   ├── novelty.py      # Novelty search (behavioral diversity)
│   ├── pareto.py       # NSGA-II multi-objective optimization
│   └── storage.py      # Evolution run persistence (PostgreSQL)
├── forum/              # Agent discussion system
│   ├── service.py      # ForumService (post/retrieve messages)
│   ├── models.py       # ForumMessage, WitnessSummary
│   ├── runner.py       # DiscussionAgentRunner (forum agent lifecycle)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0xhubed/agent-trading-arena](https://github.com/0xhubed/agent-trading-arena) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
