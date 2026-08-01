---
trigger: always_on
description: This file is the source of truth for any agent (Claude Code, Codex, others) working in this repo. `CLAUDE.md` is a symlink to this file — edit one, both update.
---

# AgentOS Railway Template

This file is the source of truth for any agent (Claude Code, Codex, others) working in this repo. `CLAUDE.md` is a symlink to this file — edit one, both update.

## Project Overview

A unified agent platform built on [Agno](https://docs.agno.com), shipped as a copy-pasteable starting point. Two reference agents demonstrate the two common shapes for supplying context to an agent. Postgres (pgvector) handles persistence for sessions, memory, and knowledge. Designed to run locally via Docker and deploy to Railway with a single script.

## Architecture

```
AgentOS  (app/main.py)
├── WebSearch  (agents/web_search.py)   — Parallel SDK or keyless MCPTools
└── CodeSearch (agents/code_search.py)  — WorkspaceContextProvider
```

Shared:
- PostgreSQL + pgvector for sessions, memory, knowledge.
- `app.settings.default_model()` returns `OpenAIResponses(id="gpt-5.4")` — bump the model in one place.
- Scheduler enabled by default (`scheduler=True`).
- Slack interface lights up automatically when both `SLACK_BOT_TOKEN` and `SLACK_SIGNING_SECRET` are set.
- JWT auth on whenever `RUNTIME_ENV == "prd"` (so production deploys are gated by default).

## Key Files

| File | Purpose |
|------|---------|
| [`app/main.py`](app/main.py) | AgentOS entrypoint — lifespan hook, conditional Slack, JWT gate. |
| [`app/settings.py`](app/settings.py) | `default_model()` factory. |
| [`app/config.yaml`](app/config.yaml) | Quick prompts per agent (keyed by agent `id`). |
| [`agents/web_search.py`](agents/web_search.py) | Reference agent — direct tools (Parallel SDK or MCP). |
| [`agents/code_search.py`](agents/code_search.py) | Reference agent — context provider. |
| [`db/session.py`](db/session.py) | `get_postgres_db()`, `create_knowledge()`. |
| [`db/url.py`](db/url.py) | Builds the database URL from env. |
| [`evals/cases.py`](evals/cases.py) | Eval cases (each is a `Case` with optional judge + reliability checks). |
| [`evals/__main__.py`](evals/__main__.py) | `python -m evals` runner — wraps agno's `AgentAsJudgeEval` + `ReliabilityEval`. |
| [`compose.yaml`](compose.yaml) | Docker Compose for local development. |
| [`railway.json`](railway.json) | Railway deploy config (Docker + 2 replicas + 4Gi/2vCPU). |

## Development Setup

### Local with Docker

```bash
cp example.env .env
# Edit .env and set OPENAI_API_KEY

docker compose up -d --build
```

Hot-reload watches `agents/`, `app/`, `db/`. Edits land in <2s. `compose.yaml` sets `RUNTIME_ENV=dev`, `AGNO_DEBUG=True`, and `WAIT_FOR_DB=True` so JWT is off and the API blocks on the DB before serving.

### Format & Validate

The format / validate / eval scripts run on the host, so they need a venv. Set one up once:

```bash
./scripts/venv_setup.sh
source .venv/bin/activate
```

Then:

```bash
./scripts/format.sh     # ruff format + import sort
./scripts/validate.sh   # ruff check + mypy (runs both, summarizes)
```

CI installs the same pinned `requirements.txt` and runs the same `scripts/validate.sh` — local and CI never drift.

## Conventions

### Agent pattern

Every agent file has the same shape:

```python
"""
<Title> Agent
=============
"""

from agno.agent import Agent

from app.settings import default_model
from db import get_postgres_db

INSTRUCTIONS = """\
<one short paragraph: what the agent does, which tools it uses, the
rules to follow when answering>
"""

my_agent = Agent(
    id="my-agent",
    name="My Agent",
    model=default_model(),
    db=get_postgres_db(),
    tools=[...],
    instructions=INSTRUCTIONS,
    enable_agentic_memory=True,
    add_datetime_to_context=True,
    add_history_to_context=True,
    num_history_runs=5,
    markdown=True,
)
```

Two patterns to copy from:

- **Direct tools** — see [`agents/web_search.py`](agents/web_search.py). The agent sees each tool individually. Best when the user knows which tools the agent needs.
- **Context provider** — see [`agents/code_search.py`](agents/code_search.py). The agent sees one `query_<thing>` tool that hands off to a sub-agent. Best for one-source agents and when collapsing many tools into one keeps the model focused.

### Database

```python
# Plain agent — sessions, memory, agentic memory live here
from db import get_postgres_db
agent_db = get_postgres_db()

# Agent with a Knowledge base (RAG) — pass through `knowledge=`
from db import create_knowledge
my_kb = create_knowledge("My Knowledge", "my_vectors")
```

Knowledge bases use PgVector with `SearchType.hybrid` and `text-embedding-3-small`. Document contents go into `<table_name>_contents`.

## Adding a new agent

Two options:

1. **Hand it to Claude Code** — paste `Run docs/create-new-agent.md` into a Claude Code session pointed at this repo. Claude asks the user what the agent should do, generates the file, registers it, smoke-tests it.
2. **Do it manually** — create `agents/<slug>.py`, register in `app/main.py`, add prompts to `app/config.yaml`. Then `docker compose restart agentos-api` — uvicorn hot-reload is unreliable for newly-registered modules, so a restart is required for the new agent to load.

## Iterating on an agent

Two recursive loops over the same agent. Use them together.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agno-agi/agentos-railway-template](https://github.com/agno-agi/agentos-railway-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
