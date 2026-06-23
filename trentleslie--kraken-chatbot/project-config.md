---
trigger: always_on
description: This file provides guidance to Claude Code when working with the KRAKEN Chatbot codebase.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with the KRAKEN Chatbot codebase.

## Commands

```bash
# Frontend (React)
npm run dev          # Start dev server (Vite HMR on port 5173)
npm run build        # Build client (Vite) → dist/
npm run check        # TypeScript type checking

# Backend (FastAPI/Python)
cd backend
uv sync              # Install/update Python dependencies
uv run uvicorn src.kestrel_backend.main:app --reload --host 127.0.0.1 --port 8000

# Testing
# Use `python -m pytest` (not bare `uv run pytest`): the venv path contains
# spaces, which breaks the console-script shebang and silently falls back to
# system Python → misleading ModuleNotFoundError. See
# docs/solutions/developer-experience/pytest-venv-path-spaces-module-invocation-2026-06-01.md
cd backend && uv run python -m pytest tests/ -v -m "not integration"
```

## LangGraph Studio (Local Development)

Visual debugger for the KRAKEN discovery pipeline.

### Setup

1. Ensure `backend/.env` exists: `cp backend/.env.example backend/.env` and fill in API keys
2. `cd backend && uv run langgraph dev`
3. Open the Studio URL printed in the terminal (requires a free [LangSmith](https://smith.langchain.com) account)
4. Graph topology renders automatically; invoke with a test query to see node execution

### Configuration

- `backend/langgraph.json` — graph entry point (`kestrel_backend.graph.builder:build_discovery_graph`) and env reference
- `backend/.env` — API keys (not committed)

### Notes

- Studio is a dev tool only — does not affect production deployment
- Hot-reloads on code changes
- Runs in-memory (no Docker/Redis/Postgres needed)
- Port 2024 by default
- **Observability**: Langfuse handles tracing, not LangSmith. `LANGSMITH_API_KEY` is not needed — ignore the Studio banner about it.

## Architecture

Full-stack application: React frontend + Python FastAPI backend communicating via WebSocket.

```
client/              React 18 SPA (Vite, Wouter router, TanStack Query, shadcn/ui + Tailwind)
backend/             FastAPI backend with Claude Agent SDK + LangGraph discovery pipeline
  src/kestrel_backend/
    main.py          FastAPI app with WebSocket endpoints
    agent.py         Claude Agent SDK integration
    kestrel_client.py    Kestrel MCP client (biomedical knowledge graph)
    graph/           LangGraph discovery pipeline (9-node workflow)
      state.py       Pydantic models and TypedDict state
      builder.py     Graph construction and routing
      runner.py      Pipeline execution and streaming
      nodes/         Individual pipeline nodes
docs/                Architecture documentation
  solutions/         Documented solutions (bugs, best practices, patterns) with YAML frontmatter
```

### Dual Operating Modes

1. **Classic Mode** — Single Claude agent with Kestrel MCP tools for direct KG queries
2. **Discovery Pipeline Mode** — 9-node LangGraph workflow for research-grade analysis

### Discovery Pipeline Nodes

`Intake → Entity Resolution → Triage → [Direct KG | Cold-Start] → Pathway Enrichment → Integration → [Temporal] → Synthesis`

| Node | Purpose |
|------|---------|
| Intake | Query parsing, entity extraction (heuristic, no LLM) |
| Entity Resolution | CURIE mapping via Kestrel MCP |
| Triage | Edge count classification (well-characterized/moderate/sparse/cold-start) |
| Direct KG | Disease associations, pathway memberships for well-characterized entities |
| Cold-Start | Analogue-based inference for sparse/unknown entities |
| Pathway Enrichment | Shared neighbors, biological themes |
| Integration | Cross-entity bridges, gap analysis |
| Temporal | Longitudinal classification (conditional) |
| Synthesis | Final report + hypothesis generation |

### Key Technical Details

- **Concurrency**: `SDK_SEMAPHORE = asyncio.Semaphore(4)` controls parallel SDK calls
- **Buffer Size**: `max_buffer_size=10MB` for large KG responses
- **Hub Detection**: Nodes with >1000 edges flagged, deprioritized in analysis
- **Two-Tier Direct KG**: Tier 1 (top entities) + Tier 2 (discovered entities) analysis

## Development Workflow

### Feature Development

1. **Create feature branch** from `main`:
   ```bash
   git checkout main && git pull origin main
   git checkout -b feat/your-feature-name
   ```

2. **Develop locally**, commit changes with descriptive messages

3. **Push and create PR**:
   ```bash
   git push -u origin feat/your-feature-name
   gh pr create --title "feat: description" --body "## Summary\n..."
   ```

4. **Wait for Greptile code review** — automated review runs on PR

5. **Address review feedback** if needed

6. **User merges PR** — merge to `main` triggers automatic deployment via GitHub Actions

### Branch Naming Conventions

| Prefix | Purpose |
|--------|---------|
| `feat/` | New features |
| `fix/` | Bug fixes |
| `refactor/` | Code refactoring |
| `perf/` | Performance improvements |
| `docs/` | Documentation updates |

## Production Deployment

### AWS Lightsail Instance

| Property | Value |
|----------|-------|
| Instance | `expert-in-the-loop-upgraded` |
| IP | `35.161.242.62` |
| URL | `https://kraken.expertintheloop.io` |
| Backend Port | 8000 |
| Service | `kraken-backend` |

### SSH Access

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trentleslie/kraken-chatbot](https://github.com/trentleslie/kraken-chatbot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-23 -->
