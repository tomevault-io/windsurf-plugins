---
trigger: always_on
description: A cryptocurrency analysis system built with Claude Code's native features: agents, MCP servers, and skills.
---

# Crypto Trading Desk - Multi-Agent Intelligence System

A cryptocurrency analysis system built with Claude Code's native features: agents, MCP servers, and skills.

## Architecture

```
User Query
    |
    v
[Claude Code - Coordinator]
    |  Reads this CLAUDE.md
    |  Routes by complexity:
    |
    +-- QUICK (1 subagent) ----------> market-monitor (haiku)
    |
    +-- STANDARD (2-3 parallel) -----> market-monitor + technical-analyst + news-sentiment
    |
    +-- FULL ANALYSIS (5 subagents) -> 3 phases with file-based coordination
         Phase 1 (parallel): market-monitor || technical-analyst || news-sentiment
         Phase 2 (sequential): risk-specialist (reads Phase 1 files)
         Phase 3 (sequential): portfolio-manager (reads all, decides)
```

## How to Use

### First-Time Setup
- `/setup` - **Run once after installing.** Detects environment, installs uv + Python deps, verifies MCP servers work. Cross-platform (macOS, Linux, Windows).

### Slash Commands
- `/quick BTC` - Fast market snapshot (1 agent, ~15 sec)
- `/analyze BTC` - Full 5-agent analysis with trading decision (~3-5 min)
- `/portfolio` - View portfolio status and open trades
- `/close-trade trade_001` - Close trade with post-mortem
- `/validate-predictions` - Review pending predictions against market data
- `/monitor` - Autonomous loop: check SL/TP, close trades, evaluate predictions
- `/create` - Extend the system with new components

### Natural Language
Ask naturally and the coordinator routes by complexity:
- "How's BTC?" → Quick (market-monitor only)
- "RSI of ETH?" → Standard (technical-analyst only)
- "Analyze SOL" → Standard (2-3 agents)
- "Full analysis of BTC with recommendation" → Full (5 agents in phases)
- "Check my portfolio" → portfolio-manager
- "Post-mortem on trade_003" → learning-agent

## Routing Rules

### How Delegation Works

All agents are spawned via the Task tool with `subagent_type: general-purpose` and an explicit `model` parameter. This ensures every subagent inherits all MCP tools from the parent conversation. The agent `.md` files in `agents/` contain the analysis framework — include "Read agents/{name}.md for your full analysis framework" in the prompt.

> **Why `general-purpose`?** Plugin-defined agent types (e.g., `crypto-trading-desk:market-monitor`) don't receive MCP tools due to a known Claude Code limitation ([#4476](https://github.com/anthropics/claude-code/issues/4476)). Using `general-purpose` with explicit `model` works in both plugin and local-clone modes.

Always include "Do NOT use the Edit tool" in every agent prompt.

### Quick Query (single subagent via Task tool)
Use for simple data points. One agent, fast response.

| Query Type | Role | Model | Key MCP tools to mention in prompt |
|-----------|------|-------|-----------------------------------|
| Price, volume, market overview | market-monitor | haiku | crypto-exchange, crypto-data, crypto-futures |
| RSI, MACD, indicators, patterns | technical-analyst | sonnet | crypto-technical, crypto-advanced-indicators |
| News, sentiment, FUD/FOMO | news-sentiment | sonnet | WebSearch, WebFetch (no MCP needed) |
| Portfolio status, trade history | portfolio-manager | opus | crypto-learning-db, crypto-exchange |
| Pattern lookup, pre-trade check | learning-agent | opus | crypto-learning-db, crypto-data |

Example delegation:
```
Task(subagent_type="general-purpose", model="haiku", prompt="You are the market-monitor agent. Read agents/market-monitor.md for your framework. Get BTC price using get_exchange_prices(symbol='BTC/USDT')... Do NOT use the Edit tool.")
```

### Standard Analysis (2-3 subagents in parallel)
Use for "analyze X" without explicit "full analysis" or "recommendation" request.
Launch 3 Tasks in parallel (all `subagent_type: general-purpose`):
- market-monitor (model: haiku) + technical-analyst (model: sonnet) + news-sentiment (model: sonnet)
- Coordinator synthesizes results

### Full Analysis with Decision (5 subagents in phases)
Use for "full analysis", "should I buy", "recommendation", or `/analyze`.
Spawn 5 subagents via the Task tool in **3 sequential phases** (see `/analyze` skill for detailed prompts):

**Phase 1 - Data Gathering (3 Tasks in parallel):**
- market-monitor (haiku) → `data/reports/YYYY-MM-DD-{symbol}/market-data.md`
- technical-analyst (sonnet) → `data/reports/YYYY-MM-DD-{symbol}/technical-analysis.md`
- news-sentiment (sonnet) → `data/reports/YYYY-MM-DD-{symbol}/news-sentiment.md`

**Wait for all 3 Task calls to return. Verify files exist on disk.**

**Phase 2 - Risk Assessment (1 Task AFTER Phase 1):**
- risk-specialist (sonnet) reads Phase 1 files → `data/reports/YYYY-MM-DD-{symbol}/risk-assessment.md`

**Wait for Task to return. Verify file exists.**

**Phase 3 - Decision (1 Task AFTER Phase 2):**
- portfolio-manager (opus) reads all files → `data/reports/YYYY-MM-DD-{symbol}/decision.md`

**IMPORTANT:** Do NOT spawn all 5 agents at once. Phase 2/3 agents MUST read files written by previous phases. Spawn them only after confirming previous phase files exist on disk.

### Special Queries (single subagent)
- "Close trade X" → portfolio-manager (close) + learning-agent (post-mortem + prediction validation)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hugoguerrap/crypto-claude-desk](https://github.com/hugoguerrap/crypto-claude-desk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
