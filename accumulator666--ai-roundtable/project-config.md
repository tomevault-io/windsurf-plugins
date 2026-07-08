---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI Mesh is a multi-AI Docker infrastructure running a Fortune 500-style AI company. Five companies (Holding, MeshTech, MeshMedia, MeshCapital, MeshVentures) each run AI roundtables with specialized participants. A CEO can pitch a project idea to any company — the system breaks it into tasks, assigns to the right specialists, executes in parallel, and synthesizes a final deliverable. A central router dispatches requests to 5 AI provider proxies, Claude Code CLI, and a remote R730 Ollama node. Redis provides cross-company messaging, PostgreSQL stores state.

## Architecture

Five-tier system: **Proxies** normalize AI APIs -> **Router** orchestrates model routing -> **Company Roundtables** run domain-scoped AI boards with project orchestration -> **Orchestrator** coordinates cross-company projects -> **Supporting services** (Redis, PostgreSQL, n8n, Caddy).

> **Migration note:** MeshCorp HQ (8160) is the next-generation replacement for the company roundtables (8130-8134) and legacy collab-chat (8139). It provides a unified dashboard with dynamic org templates instead of fixed per-company containers. Both old and new systems coexist during migration.

```
Client / Shell
    |
Caddy (reverse proxy, TLS — ports 80/443/8444)
    |
AI Router (8110) — central orchestrator
    |--- Claude Proxy (8100) --- Anthropic API
    |--- ChatGPT Proxy (8101) --- OpenAI API
    |--- Grok Proxy (8102) --- xAI API
    |--- Gemini Proxy (8103) --- Google API
    |--- DeepSeek Proxy (8105) --- DeepSeek API
    |--- Claude Code API (8104) --- Claude CLI
    '--- R730 Ollama (10.0.0.2:11434)

MeshCorp HQ (8160) — unified dashboard, dynamic org templates

Company Roundtables (8130-8134) — Fortune 500 simulation
    |--- Holding Board (8130) — CEO, orchestrates cross-company projects
    |--- MeshTech (8131) — Engineering
    |--- MeshMedia (8132) — Marketing & Content
    |--- MeshCapital (8133) — Finance (+ Options Trader integration)
    '--- MeshVentures (8134) — R&D & Sales

Shared Infrastructure:
    PostgreSQL (ai_mesh DB via ai-stack_default network)
    Redis (pub/sub, cross-company comms)
    CrewAI Agents (8120) — autonomous background jobs
    Legacy Collab Chat (8139) — being replaced by company roundtables
    n8n (5678) — workflow automation
```

### Key Directories

- **`router/`** — Central orchestrator. `routing.py` is **source of truth** for all model->proxy mappings (40+ models), R730 model set, aliases, and auto-routing patterns.
- **`proxies/{claude,chatgpt,grok,gemini,deepseek}/`** — Thin FastAPI wrappers normalizing each provider API to OpenAI-compatible `/v1/chat/completions`. Each has `main.py` + `requirements.txt`.
- **`companies/shared/`** — Shared code for all company roundtables:
  - `base_app.py` — `create_company_app()` factory with roundtable engine, project mode, WebSocket handler
  - `orchestrator.py` — Cross-company project orchestration, `SKILL_ROUTING` (35 skill->company mappings)
  - `db.py` — Async PostgreSQL (asyncpg), auto-creates all tables on first connect
  - `company_comms.py` — Redis pub/sub for cross-company messaging
- **`companies/{holding,meshtech,meshmedia,meshcapital,meshventures}/`** — Each has `main.py` (calls `create_company_app()`), `participants.json`, `company_config.json`, `index.html`. The `shared/` symlink in each points to `companies/shared/`.
- **`meshcorp/`** — MeshCorp HQ: unified dashboard replacing company roundtables. `main.py` (FastAPI), `conversation.py` (event-driven engine), `projects.py` (lifecycle manager), `routes.py` (REST + WebSocket API), `templates/` (7 org template JSONs), `frontend/` (Svelte SPA).
- **`collab-chat/`** — Legacy monolithic roundtable (single `main.py` ~2900 lines + `index.html`). Has its own `CLAUDE.md` with detailed docs. Being replaced by MeshCorp HQ.
- **`agents/`** — CrewAI background agents. Has its own `Dockerfile`. `crew.py` defines 5 agents (strategist, researcher, builder, marketer, finance).
- **`claude-code/`** — REST wrapper for Claude Code CLI. Has its own `Dockerfile`. Uses OAuth credentials from `/root/.claude/.credentials.json`.
- **`claude-mcp/`** — MCP (Model Context Protocol) server providing delegation tools to cheaper/free models.
- **`shared/`** — Top-level shared utilities: `models.py` (Pydantic models for chat protocol), `config.py` (legacy proxy URL mappings, superseded by `router/routing.py`).

## Commands

```bash
# Start everything
docker compose up -d

# Check health of all backends
curl -s http://localhost:8110/health | jq

# Check all companies
for p in 8130 8131 8132 8133 8134; do curl -s http://localhost:$p/health | jq .company_name; done

# View logs
docker logs ai-mesh-router -f --tail 50

# Restart a service (picks up code changes — volumes are mounted)
docker restart ai-mesh-router

# Restart all company roundtables after editing companies/shared/ code
docker restart ai-mesh-holding-board ai-mesh-meshtech ai-mesh-meshmedia ai-mesh-meshcapital ai-mesh-meshventures

# Only claude-code and crewai-agents need docker compose build (they have Dockerfiles)
docker compose build claude-code && docker compose up -d claude-code

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [accumulator666/ai-roundtable](https://github.com/accumulator666/ai-roundtable) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
