---
trigger: always_on
description: This project implements a multi-agent AI system for financial analysis and earnings research. All development must adhere to these conventions. Also see `CLAUDE.md` for detailed architecture and `docs/` for runbooks.
---

# earnings-edge: Agent Development Guidelines

This project implements a multi-agent AI system for financial analysis and earnings research. All development must adhere to these conventions. Also see `CLAUDE.md` for detailed architecture and `docs/` for runbooks.

> **Accuracy note**: sections marked *(planned — not implemented)* describe intended design whose
> modules do not exist yet. Everything else reflects code on disk. If you build or move a module,
> update this file **and** `AGENTS.md` (they are near-duplicates) in the same change.

---

## Architecture & Core Concepts

### The Graph

- **Orchestration**: LangGraph StateGraph with nodes = agents, edges = handoffs
- **Topology** (`src/agentic_app/orchestration/graph.py`): a fixed pipeline, *not* a supervisor/router:

  ```text
  ingest --> {sentiment, kpi}      (parallel fan-out)
  {sentiment, kpi} --> synthesize  (fan-in: waits for BOTH)
  synthesize --> evaluate
  evaluate --(route_after_eval)--> revise --> evaluate   (self-correction loop)
                                \-> deliver --> END
  ```

- **State** (`src/agentic_app/orchestration/state.py`): `EarningsState` TypedDict — ticker/year/quarter,
  transcripts, `sentiment` + `kpis` (both `Annotated[..., operator.add]` so parallel branches merge),
  `signal`, `brief_markdown`, `grounding_score`, `revision_count`, `delivery_log`, `errors`.
  Pydantic models `EarningsKPIs` + `SurpriseSignal` live here too.
- **Persistence**: selected at runtime in `src/agentic_app/main.py` (there is **no** `checkpoints.py`).
  `DATABASE_URL` starting with `postgres` → `PostgresSaver` + `PostgresStore` (`.setup()` creates tables);
  otherwise local `SqliteSaver` (`earningsedge.db`) + `InMemoryStore`. Unreachable Postgres falls back to
  SQLite with a warning. `thread_id` is `f"{ticker}-{year}-Q{quarter}"`.

### Agent Roles

One module per node in `src/agentic_app/agents/`, each a **sync** `*_node(state) -> dict`:

- **Ingestion** (`ingestion_agent.py`): fetches transcript / press-release text into state
- **Sentiment** (`sentiment_agent.py`): tone analysis; reads the Store (injected automatically)
- **KPI** (`kpi_agent.py`): extracts structured `EarningsKPIs` (revenue, EPS, guidance, segments)
- **Synthesis** (`synthesis_agent.py`): fuses sentiment + KPIs into `signal` + `brief_markdown`
- **Evaluation** (`evaluation_agent.py`): scores grounding; `route_after_eval` returns `revise` or `deliver`
- **Delivery** (`delivery_agent.py`): final output; reads + writes the Store

`base.py` provides `BaseAgent`. Sentiment and KPI run **concurrently** — never assume one sees the other's output.

### Tools & Skills Boundary

- **Tools** (`src/agentic_app/tools/`): `registry.py` (`ToolRegistry`) + Pydantic schemas in `schemas.py`
- **Skills** (`skills/`): Multi-step workflows with bundled instructions + assets; agents load dynamically.
  Real skills are domain-specific: `kpi-extraction`, `finbert-tone-analysis`, `sec-edgar-8k-retrieval`,
  `signal-synthesis-scoring`, `grounding-faithfulness-eval`, `composio-delivery`, and others.
- **MCP** *(planned — not implemented)*: `src/agentic_app/mcp/` is a stub (`__init__.py` only).
  The venv does ship `edgartools-mcp` as an external server.

### RAG Pipeline *(planned — not implemented)*

`src/agentic_app/rag/` is a stub (`__init__.py` only). No ingest, chunking, embeddings, retriever, or
vectorstore module exists, despite `VECTORDB_*` / `CHUNK_*` / `RETRIEVAL_*` vars in `.env.example`, a
`qdrant` service in `docker-compose.yml`, and `scripts/seed_vectorstore.py`. Intended design:
structure-aware chunk → embed → upsert; hybrid search + rerank behind a swappable pgvector/Qdrant Protocol.

---

## Code Style & Typing

### Python

- **All files**: Python 3.11+ (`requires-python = ">=3.11"`; the venv is 3.12), strict typing (`from __future__ import annotations`)
- **Sync, not async**: graph nodes are plain `def`. There is no async code in the agent path today — do
  not introduce `async def` nodes without changing how the graph is invoked.
- **Functions**: Type-annotated arguments and returns; use `Pydantic` for complex inputs/outputs
- **Error handling**: accumulate failures into `state["errors"]` (an `operator.add` list) rather than
  raising through a node; typed exceptions at tool boundaries
- **Imports**: src-layout — `packages = ["src/agentic_app"]`; import absolutely (`from agentic_app.x import y`); organize stdlib → third-party → local

### Naming

- **Agents**: `{role}_agent.py` exposing `{role}_node` (e.g., `kpi_agent.py` → `kpi_node`)
- **Tools**: `{capability}.py`
- **Skills**: `{skill_domain}/SKILL.md` + `{skill_domain}/scripts/`, `{skill_domain}/resources/`
- **Prompts**: `prompts/{role}/v{N}.md` (immutable; version in `prompts/registry.yaml`).
  Roles: `ingestion`, `kpi`, `sentiment`, `synthesis`, `evaluation`, `delivery`, `shared`.

### Linting & Format

- **Ruff**: `check` + `format`; configured in `pyproject.toml`
- **MyPy**: `strict = true`; see `pyproject.toml`
- **Pre-commit**: ruff-check, ruff-format, mypy, gitleaks, prompt-consistency (`.pre-commit-config.yaml`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [suhaas/earnings-edge](https://github.com/suhaas/earnings-edge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
