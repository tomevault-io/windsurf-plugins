---
trigger: always_on
description: Dash is a self-learning data agent that delivers **insights, not just SQL results**. It uses a team of specialists (Analyst + Engineer) coordinated by a leader to handle data queries and build computed data assets. Built on [Agno](https://docs.agno.com). Runs in Slack, the terminal, or the [AgentOS](https://os.agno.com) web UI.
---

# CLAUDE.md

## Project Overview

Dash is a self-learning data agent that delivers **insights, not just SQL results**. It uses a team of specialists (Analyst + Engineer) coordinated by a leader to handle data queries and build computed data assets. Built on [Agno](https://docs.agno.com). Runs in Slack, the terminal, or the [AgentOS](https://os.agno.com) web UI.

## Structure

```
dash/
├── team.py               # Dash team (leader, coordinate mode)
├── settings.py            # Shared config (DB, model, knowledge bases, Slack)
├── instructions.py        # Instruction builders per agent role
├── paths.py               # Path constants
├── __main__.py            # CLI entry point (python -m dash)
├── agents/
│   ├── analyst.py         # SQL queries, data analysis, insights (read-only)
│   └── engineer.py        # Views, summary tables, computed data (dash schema)
├── context/               # Runtime prompt builders (reads knowledge/)
│   ├── semantic_model.py  # Table metadata → system prompt
│   └── business_rules.py  # Business rules → system prompt
└── tools/
    ├── build.py           # Tool assembly per agent role (schema boundaries)
    ├── introspect.py      # Runtime schema inspection (both schemas)
    ├── save_query.py      # Save validated queries to knowledge
    └── update_knowledge.py # Record schema changes to knowledge

knowledge/                 # Data files loaded into vector DB (1:1 mapping)
├── tables/                # Table metadata JSON files (SaaS metrics)
├── queries/               # Validated SQL query patterns
└── business/              # Business rules, metrics, gotchas

app/
├── main.py               # AgentOS entry point (teams, scheduler, Slack interface)
└── config.yaml            # Quick prompts

db/
├── __init__.py            # Re-exports: db_url, get_postgres_db, get_sql_engine, etc.
├── session.py             # PostgreSQL + PgVector + dual schema (public/dash)
└── url.py                 # Database URL builder

evals/                     # Evaluation framework (Agno eval classes)
├── run.py                 # Unified eval runner
└── cases/                 # Test cases by category
    ├── accuracy.py        # AccuracyEval — data correctness
    ├── routing.py         # ReliabilityEval — team routes correctly
    ├── security.py        # AgentAsJudgeEval — no credential leaks
    ├── governance.py      # AgentAsJudgeEval — refuses destructive SQL
    └── boundaries.py      # AgentAsJudgeEval — schema access boundaries

scripts/
├── generate_data.py       # Generate SaaS sample data
├── load_knowledge.py      # Load knowledge into vector DB
├── venv_setup.sh          # Create virtualenv (uses uv)
├── format.sh              # ruff format + import sorting
├── validate.sh            # ruff check + mypy
├── generate_requirements.sh # uv pip compile → requirements.txt
├── build_image.sh         # Multi-platform Docker build
├── entrypoint.sh          # Docker entrypoint (DB wait, banner)
├── railway_up.sh          # First-time Railway setup
├── railway_redeploy.sh    # Redeploy to Railway
└── railway_env.sh         # Sync .env.production to Railway

docs/
├── SLACK_CONNECT.md       # Slack app setup guide with manifest
└── TEST_QUESTIONS.md      # Manual test questions (routing, data quality, edge cases)
```

## Commands

```bash
./scripts/venv_setup.sh && source .venv/bin/activate
./scripts/format.sh      # Format code (ruff format + isort)
./scripts/validate.sh    # Lint + type check (ruff + mypy)
python -m dash           # CLI mode
python -m dash.team      # Test mode (runs sample queries)

# Data & Knowledge
python scripts/generate_data.py      # Generate SaaS sample data
python scripts/load_knowledge.py     # Load knowledge into vector DB

# Evaluations
python -m evals                      # Run all evals
python -m evals --category accuracy  # Run specific category
python -m evals --verbose            # Show response details

# Deployment (uses .env.production)
./scripts/railway_up.sh              # First-time Railway setup
./scripts/railway_redeploy.sh        # Redeploy
./scripts/railway_env.sh             # Sync .env.production to Railway

# Dependencies
./scripts/generate_requirements.sh           # Regenerate requirements.txt
./scripts/generate_requirements.sh upgrade   # Regenerate with latest versions
```

## Architecture

**Dual Schema:**

| Schema | Owner | Access |
|--------|-------|--------|
| `public` | Company (loaded externally) | Read-only — never modified by agents |
| `dash` | Engineer agent | Views, summary tables, computed data |

**Team (Coordinate Mode):**

| Agent | Role | Tools | Schema Access |
|-------|------|-------|---------------|
| **Dash (Leader)** | Routes requests, synthesizes insights | SlackTools (optional) | — |
| **Analyst** | SQL queries, data analysis, insights | SQLTools (read-only, enforced by DB), introspect_schema, save_validated_query, ReasoningTools | Reads public + dash |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agno-agi/dash](https://github.com/agno-agi/dash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
