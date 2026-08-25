---
trigger: always_on
description: This file provides guidance to Claude Code and Copilot when working on the ClawGold XAUUSD trading system codebase.
---

# CLAUDE.md — ClawGold Development Guide

This file provides guidance to Claude Code and Copilot when working on the ClawGold XAUUSD trading system codebase.

## Project Overview

**ClawGold** is an AI-powered autonomous gold (XAUUSD) trading system with:
- **Multi-AI Agent System** — Dispatch tasks to OpenCode, KiloCode, Gemini, Codex via CLI (no model training)
- **Sub-Agent Orchestration** — 5 domain-specific roles (researcher, analyst, strategist, monitor, general)
- **Automated Scheduling** — Background daemon for daily/periodic trading workflows
- **LLM Observability** — Langfuse integration for tracing, cost tracking, quality evaluation
- **MetaTrader 5 Integration** — Direct Python MT5 API for trading execution
- **News & Sentiment Engine** — Multi-source research with AI consensus

**Repository:** `d:\Projects\Github\ClawGold`  
**Language:** Python 3.10+ (TypeScript ESM for future components)  
**Package Manager:** pip (standard Python)

---

## Development Commands

### Essential Setup & Build
```bash
# Install dependencies
pip install -r requirements.txt

# Run tests
python -m unittest discover -s test -p "test_*.py" -v

# Validate configuration
python claw.py validate

# Check balance (MT5 connection test)
python claw.py balance
```

### Testing
```bash
# All tests
python -m unittest discover -s test -p "test_*.py" -v

# Specific test file
python -m unittest test.test_agent_system -v

# With coverage
python -m coverage run -m unittest discover -s test && python -m coverage report

# Watch mode (rerun on changes)
while True; do python -m unittest discover -s test; sleep 2; done
```

### Development Workflow
```bash
# Start development
python claw.py agent tools      # List available AI CLI tools
python claw.py agent run gemini "Test prompt"

# Check logs
tail -f logs/trades.log

# Run a specific command
python claw.py agent daily      # Daily routine
python claw.py agent history    # View execution history
python claw.py agent metrics    # Per-tool performance

# Schedule debug
python claw.py agent schedule status
python claw.py agent schedule log
```

---

## High-Level Architecture

### Core Components

**1. Agent System** (`scripts/agent_*.py` — 3 modules)
- `agent_executor.py` — Unified CLI tool interface (tool discovery, execution, caching, metrics)
- `sub_agent.py` — Role-based orchestrator (5 roles: researcher, analyst, strategist, monitor, general)
- `agent_scheduler.py` — Background task scheduler (daily, interval, cron patterns)
- `langfuse_tracer.py` — LLM observability integration

**2. Trading Core** (`scripts/`)
- `mt5_manager.py` — MT5 connection handler
- `risk_manager.py` — Position sizing, daily loss limits, margin checks
- `advanced_trader.py` — Trading strategies (grid, breakout, scalping, etc.)
- `position_monitor.py` — Real-time position alerts

**3. News & Sentiment** (`scripts/`)
- `news_aggregator.py` — Multi-source news collection
- `news_db.py` — SQLite schema for news storage
- `ai_researcher.py` — AI tool integration (parallel requests)
- `sentiment_analyzer.py` — Keyword-based sentiment scoring

**4. CLI Surface** (`claw.py`)
- Commander-based CLI with subcommands
- ~2000 lines, integrates all components
- Agent commands: `claw.py agent <subcommand>`

**5. Supporting** (`scripts/`)
- `config_loader.py` — Load config.yaml with env overrides
- `config_validator.py` — TypeBox schema validation
- `trade_journal.py` — Trade analytics and journaling
- `logger.py` — Unified logging (~trades.log)
- `notifier.py` — Telegram notifications

### Data Flow

```
Market Event / Schedule Trigger
    ↓
SubAgent Orchestrator (pick role)
    ↓
AgentExecutor (dispatch to AI CLI tool)
    ├─ Tool Discovery
    ├─ Try Cache
    ├─ Execute subprocess
    ├─ Record History/Metrics
    └─ Trace to Langfuse
    ↓
AI CLI Tool Response
    ↓
Consensus/Analysis
    ↓
Trade Signal (BUY/SELL/HOLD)
    ↓
Risk Manager (position size, margin check)
    ↓
MT5 Execution
    ↓
Position Monitor → Telegram Alert
```

### Key Subsystems

**Session Model:**
- Single main session for CLI trading
- Sub-agent tasks are stateless (no session needed)

**Routing:**
- CLI agents route tasks based on role and preferred tool
- Fallback chain if tool unavailable

**Configuration:**
- JSON-based `config.yaml`
- Env var overrides (MT5_LOGIN, MT5_PASSWORD, etc.)
- Agent settings: preferred tools, cache TTL, scheduler config, Langfuse

**Security:**
- MT5 credentials in env vars or `.env` file
- Langfuse API keys in env or config (optional)
- Trade execution requires explicit confirmation (optional flag)

**Storage:**
- SQLite: `data/agent_history.db` (execution history)
- SQLite: `data/agent_scheduler.db` (task state)
- SQLite: `data/news.db` (news & sentiment)
- SQLite: `data/clawgold.db` (trade journal)
- JSON cache: `data/agent_cache/` (prompt responses)
- Logs: `logs/trades.log` (unified logging)

---

## Key Files & Entry Points

| File | Purpose | Lines |
|------|---------|-------|
| `claw.py` | Main CLI entry point | ~1971 |
| `scripts/agent_executor.py` | AI CLI executor (tool discovery, execution, cache, metrics) | ~641 |
| `scripts/sub_agent.py` | Sub-agent orchestrator (5 roles) | ~642 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenKrab/ClawGold](https://github.com/OpenKrab/ClawGold) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
