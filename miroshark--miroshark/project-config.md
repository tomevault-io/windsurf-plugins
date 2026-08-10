---
trigger: always_on
description: Guidance for AI coding agents (Claude Code and similar) working in this repo. Human-facing docs live in [`README.md`](.github/README.md) and [`docs/`](docs/) — this file is the agent's map: how the code is laid out, how to run and test it, and the conventions a change must respect to pass CI and review.
---

# CLAUDE.md

Guidance for AI coding agents (Claude Code and similar) working in this repo. Human-facing docs live in [`README.md`](.github/README.md) and [`docs/`](docs/) — this file is the agent's map: how the code is laid out, how to run and test it, and the conventions a change must respect to pass CI and review.

## What MiroShark is

**Simulate anything, for $1 & less than 10 min.** Drop in a document (or just a question) and it spawns hundreds of grounded agents that post, argue, and trade across Twitter, Reddit, and a prediction market hour-by-hour, then writes a report citing what actually happened. One simulation runs for ~$1 in under 10 minutes. License is **AGPL-3.0** — keep contributions compatible.

The simulation pipeline has five phases: **graph build → agent setup → simulation → report → interaction**. See [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) for the engine, the cross-platform data flow, and the graph-memory ingestion/retrieval stack — read it before touching the simulation or memory code.

## Repo layout

```
backend/                 Python 3.11+ Flask backend (uv-managed)
  run.py                 Entry point — validates Config, then create_app()
  app/
    __init__.py          Flask application factory: blueprint registration + auth guard
    config.py            Config.validate() — env-driven settings & feature flags
    api/                 HTTP blueprints, one file per concern (simulation, report, feed, …)
    services/            Business logic (~60 modules: report_agent, polymarket_service, …)
    storage/             Neo4j graph-memory stack (search, rerank, entity resolution, …)
    prompts/ models/ countries/ preset_templates/
  wonderwall/            Bundled simulation engine (camel-ai fork): social_agent, social_platform, clock, environment, simulations
  cli.py                 Thin HTTP client for a running backend (argparse + urllib only)
  mcp_server.py          stdio MCP server exposing graph queries to Claude Desktop / Cursor
  openapi.yaml           Source of truth for the HTTP surface (drift-tested against routes)
  tests/                 pytest suite (test_unit_*, test_integration_*, test_smoke_*)
frontend/                Vue 3 + Vite SPA (axios, vue-router, d3)
docs/                    Deep-dive docs (+ *.zh-CN.md translations)
miroshark                Bash launcher: checks deps, starts Neo4j, installs + serves both apps
```

## Setup, run, test

Prereqs: **Python 3.11+**, **Node 18+** (engines pin **Node ≥22**), **Neo4j**, and at least one LLM key (default lineup targets [OpenRouter](https://openrouter.ai/)). `cp .env.example .env` and fill the `*_API_KEY` slots — note the **same OpenRouter key goes in 5 places** (`LLM_API_KEY`, `SMART_API_KEY`, `NER_API_KEY`, `OPENAI_API_KEY`, `EMBEDDING_API_KEY`). Every variable is documented in [`docs/CONFIGURATION.md`](docs/CONFIGURATION.md).

```bash
npm run setup:all        # npm install + frontend deps + (cd backend && uv sync)
docker compose up -d neo4j   # NEO4J_PASSWORD must be set in .env first
npm run dev              # backend :5001 + frontend :3000 (predev frees stale ports)
./miroshark             # alternative: launcher does deps + Neo4j + both servers
```

Run the **fast offline unit suite before pushing** — it's the same gate CI runs:

```bash
cd backend && pytest -m "not integration"        # fast, offline, no Neo4j/LLM
pytest -m integration                            # endpoint contracts (needs live :5001)
pytest -m "integration and slow"                 # full-pipeline smoke (minutes)
```

Markers (`pytest.ini`): `integration` needs a live backend at `$MIROSHARK_API_URL`, `slow` is multi-minute, `neo4j` needs a live graph DB. Unit tests must stay **offline** — no live Flask app, no Neo4j — so they run in the bare CI environment.

## CI gates (`.github/workflows/tests.yml`)

A PR to `main` must pass three jobs:
1. **Backend unit tests** — `pytest -m "not integration"` on Python 3.11 with a *thin* dependency set (no torch/transformers). Don't make a unit test import heavy ML deps.
2. **Frontend build** — `npm run build` (Vite) in `frontend/`.
3. **Camel agent smoke test** — installs real `camel-ai` + torch and runs `tests/test_smoke_camel_agent.py`; it's the only job exercising the camel ↔ wonderwall loop, so a camel-ai bump that breaks the agent loop fails here instead of shipping a zero-action simulation.

## Conventions that matter

- **Adding an HTTP endpoint is contract-first.** `backend/tests/test_unit_openapi.py` fails CI if `openapi.yaml` and the real Flask routes disagree. To add one: register the route on the right blueprint in `app/api/`, document the path in `openapi.yaml` under a declared tag, and add an offline `test_unit_<feature>.py`. A brand-new blueprint must also be registered in `app/__init__.py` and added to the drift test's prefix map. Internal/debug routes go on the test's undocumented allowlist instead. Full recipe: [`CONTRIBUTING.md`](.github/CONTRIBUTING.md#adding-an-api-endpoint).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MiroShark/MiroShark](https://github.com/MiroShark/MiroShark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
