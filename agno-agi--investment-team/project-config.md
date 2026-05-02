---
trigger: always_on
description: This file provides context for Claude Code when working with this repository.
---

# CLAUDE.md

This file provides context for Claude Code when working with this repository.

## Project Overview

Agentic Investment Team — A multi-agent system built with Agno that simulates a professional investment team deploying $10M into public equities. Demonstrates 5 multi-agent architectures, three-layer knowledge, and institutional learning.

## Architecture

```
AgentOS (app/main.py)
├── Agents (7 reusable specialists)
│   ├── Market Analyst          — macro trends, sector analysis, news (Exa + YFinance)
│   ├── Financial Analyst       — fundamentals, valuation, financials (YFinance)
│   ├── Technical Analyst       — price action, momentum, entry/exit (YFinance)
│   ├── Risk Officer            — downside scenarios, portfolio exposure (YFinance)
│   ├── Knowledge Agent         — research library (RAG) + memo archive (FileTools)
│   ├── Memo Writer             — synthesizes analysis into formal memos (FileTools)
│   └── Committee Chair         — final decisions, capital allocation (Gemini 3.1 Pro)
│
├── Teams (4 architectures)
│   ├── Coordinate Team         — Chair orchestrates analysts dynamically
│   ├── Route Team              — routes questions to the right specialist
│   ├── Broadcast Team          — all analysts evaluate simultaneously
│   └── Task Team               — autonomous task decomposition
│
├── Workflows (1 architecture)
│   └── Investment Workflow     — deterministic pipeline with parallel steps
│
├── Three-Layer Knowledge
│   ├── Layer 1: Static Context     — mandate, risk policy, process (always in prompt)
│   ├── Layer 2: Research Library   — company profiles, sector analysis (PgVector RAG)
│   └── Layer 3: Memo Archive       — past investment memos (FileTools)
│
└── Institutional Learning          — patterns, corrections, insights (LearningMachine)
```

All specialist agents use:
- Gemini 3 Flash model (`gemini-3-flash-preview`)
- PostgreSQL database (pgvector) for persistence
- Committee context (Layer 1) in system prompt
- Shared knowledge base (Layer 2) for RAG
- Shared learnings (institutional learning)

Committee Chair and team leaders use:
- Gemini 3.1 Pro model (`gemini-3.1-pro-preview`)

## Key Files

| File | Purpose |
|------|---------|
| `app/main.py` | AgentOS entry point, registers all agents/teams/workflows |
| `app/load_knowledge.py` | Loads research documents into PgVector (run once) |
| `app/config.yaml` | Quick prompts for each agent, team, and workflow |
| `agents/settings.py` | Shared instances: knowledge, learnings, paths, URLs |
| `agents/market_analyst.py` | Market Analyst — Exa MCP + YFinance |
| `agents/financial_analyst.py` | Financial Analyst — YFinance |
| `agents/technical_analyst.py` | Technical Analyst — YFinance |
| `agents/risk_officer.py` | Risk Officer — YFinance |
| `agents/knowledge_agent.py` | Knowledge Agent — RAG + FileTools |
| `agents/memo_writer.py` | Memo Writer — FileTools (save) |
| `agents/committee_chair.py` | Committee Chair — Gemini 3.1 Pro, no tools |
| `teams/coordinate_team.py` | Coordinate team (dynamic orchestration) |
| `teams/route_team.py` | Route team (single dispatch) |
| `teams/broadcast_team.py` | Broadcast team (parallel evaluation) |
| `teams/task_team.py` | Task team (autonomous decomposition) |
| `workflows/investment_workflow.py` | Deterministic 5-step pipeline |
| `context/loader.py` | Loads `*.md` context files into `COMMITTEE_CONTEXT` |
| `context/mandate.md` | Fund rules: $10M, public equities, limits |
| `context/risk_policy.md` | Position sizing, portfolio risk, rebalancing |
| `context/process.md` | 6-step evaluation pipeline |
| `db/session.py` | `get_postgres_db()` and `create_knowledge()` helpers |
| `db/url.py` | Builds database URL from environment |
| `compose.yaml` | Local development with Docker |

## Development Setup

### Virtual Environment

```bash
./scripts/venv_setup.sh
source .venv/bin/activate
```

### Format & Validation

```bash
source .venv/bin/activate && ./scripts/format.sh
source .venv/bin/activate && ./scripts/validate.sh
```

## Conventions

### Agent Pattern

All agents follow this structure:

```python
from agno.agent import Agent
from agno.models.google import Gemini
from agno.learn import LearningMachine, LearnedKnowledgeConfig, LearningMode

from context import COMMITTEE_CONTEXT
from agents.settings import team_knowledge, team_learnings
from db import get_postgres_db

agent_db = get_postgres_db()

instructions = f"""\
You are the [Role] on a $10M investment team.

## Committee Rules (ALWAYS FOLLOW)

{COMMITTEE_CONTEXT}

## Your Role
[Role-specific instructions...]
"""

my_agent = Agent(
    id="my-agent",
    name="My Agent",
    model=Gemini(id="gemini-3-flash-preview"),
    db=agent_db,
    instructions=instructions,
    tools=[...],
    knowledge=team_knowledge,
    search_knowledge=True,
    learning=LearningMachine(
        knowledge=team_learnings,
        learned_knowledge=LearnedKnowledgeConfig(
            mode=LearningMode.AGENTIC,
            namespace="global",
        ),
    ),
    add_datetime_to_context=True,
    add_history_to_context=True,
    num_history_runs=5,
    markdown=True,
    enable_agentic_memory=True,
)
```

### Critical Rules

1. **Never duplicate knowledge instances** — always import from `agents.settings`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agno-agi/investment-team](https://github.com/agno-agi/investment-team) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
