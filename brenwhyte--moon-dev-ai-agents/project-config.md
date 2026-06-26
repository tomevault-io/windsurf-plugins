---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an experimental AI trading system that orchestrates 48+ specialized AI agents to analyze markets, execute strategies, and manage risk across cryptocurrency markets (primarily Solana). The project uses a modular agent architecture with unified LLM provider abstraction supporting Claude, GPT-4, DeepSeek, Groq, Gemini, and local Ollama models.

## Key Development Commands

### Environment Setup
```bash
# Use existing conda environment (DO NOT create new virtual environments)
conda activate tflow

# Install/update dependencies
pip install -r requirements.txt

# IMPORTANT: Update requirements.txt every time you add a new package
pip freeze > requirements.txt
```

### Running the System
```bash
# Run main orchestrator (controls multiple agents)
python src/main.py

# Run individual agents standalone
python src/agents/trading_agent.py
python src/agents/risk_agent.py
python src/agents/rbi_agent.py
python src/agents/chat_agent.py
# ... any agent in src/agents/ can run independently
```

### Backtesting
```bash
# Use backtesting.py library with pandas_ta or talib for indicators
# Sample OHLCV data available at:
# /Users/md/Dropbox/dev/github/moon-dev-ai-agents-for-trading/src/data/rbi/BTC-USD-15m.csv
```

## Architecture Overview

### Core Structure
```
src/
├── agents/              # 48+ specialized AI agents (each <800 lines)
├── models/              # LLM provider abstraction (ModelFactory pattern)
├── strategies/          # User-defined trading strategies
├── scripts/             # Standalone utility scripts
├── data/                # Agent outputs, memory, analysis results
├── config.py            # Global configuration (positions, risk limits, API settings)
├── main.py              # Main orchestrator for multi-agent loop
├── nice_funcs.py        # ~1,200 lines of shared trading utilities
├── nice_funcs_hl.py     # Hyperliquid-specific utilities
└── ezbot.py             # Legacy trading controller
```

### Agent Ecosystem

**Trading Agents**: `trading_agent`, `strategy_agent`, `risk_agent`, `copybot_agent`
**Market Analysis**: `sentiment_agent`, `whale_agent`, `funding_agent`, `liquidation_agent`, `chartanalysis_agent`
**Content Creation**: `chat_agent`, `clips_agent`, `tweet_agent`, `video_agent`, `phone_agent`
**Strategy Development**: `rbi_agent` (Research-Based Inference - codes backtests from videos/PDFs), `research_agent`
**Specialized**: `sniper_agent`, `solana_agent`, `tx_agent`, `million_agent`, `tiktok_agent`, `compliance_agent`

Each agent can run independently or as part of the main orchestrator loop.

### LLM Integration (Model Factory)

Located at `src/models/model_factory.py` and `src/models/README.md`

**Unified Interface**: All agents use `ModelFactory.create_model()` for consistent LLM access
**Supported Providers**: Anthropic Claude (default), OpenAI, DeepSeek, Groq, Google Gemini, Ollama (local)
**Key Pattern**:
```python
from src.models.model_factory import ModelFactory

model = ModelFactory.create_model('anthropic')  # or 'openai', 'deepseek', 'groq', etc.
response = model.generate_response(system_prompt, user_content, temperature, max_tokens)
```

### Configuration Management

**Primary Config**: `src/config.py`
- Trading settings: `MONITORED_TOKENS`, `EXCLUDED_TOKENS`, position sizing (`usd_size`, `max_usd_order_size`)
- Risk management: `CASH_PERCENTAGE`, `MAX_POSITION_PERCENTAGE`, `MAX_LOSS_USD`, `MAX_GAIN_USD`, `MINIMUM_BALANCE_USD`
- Agent behavior: `SLEEP_BETWEEN_RUNS_MINUTES`, `ACTIVE_AGENTS` dict in `main.py`
- AI settings: `AI_MODEL`, `AI_MAX_TOKENS`, `AI_TEMPERATURE`

**Environment Variables**: `.env` (see `.env_example`)
- Trading APIs: `BIRDEYE_API_KEY`, `MOONDEV_API_KEY`, `COINGECKO_API_KEY`
- AI Services: `ANTHROPIC_KEY`, `OPENAI_KEY`, `DEEPSEEK_KEY`, `GROQ_API_KEY`, `GEMINI_KEY`
- Blockchain: `SOLANA_PRIVATE_KEY`, `HYPER_LIQUID_ETH_PRIVATE_KEY`, `RPC_ENDPOINT`

### Shared Utilities

**`src/nice_funcs.py`** (~1,200 lines): Core trading functions
- Data: `token_overview()`, `token_price()`, `get_position()`, `get_ohlcv_data()`
- Trading: `market_buy()`, `market_sell()`, `chunk_kill()`, `open_position()`
- Analysis: Technical indicators, PnL calculations, rug pull detection

**`src/agents/api.py`**: `MoonDevAPI` class for custom Moon Dev API endpoints
- `get_liquidation_data()`, `get_funding_data()`, `get_oi_data()`, `get_copybot_follow_list()`

### Data Flow Pattern

```
Config/Input → Agent Init → API Data Fetch → Data Parsing →
LLM Analysis (via ModelFactory) → Decision Output →
Result Storage (CSV/JSON in src/data/) → Optional Trade Execution
```

## Development Rules

### File Management
- **Keep files under 800 lines** - if longer, split into new files and update README
- **DO NOT move files without asking** - you can create new files but no moving
- **NEVER create new virtual environments** - use existing `conda activate tflow`
- **Update requirements.txt** after adding any new package

### Backtesting
- Use `backtesting.py` library (NOT their built-in indicators)
- Use `pandas_ta` or `talib` for technical indicators instead

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brenwhyte/moon-dev-ai-agents](https://github.com/brenwhyte/moon-dev-ai-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
