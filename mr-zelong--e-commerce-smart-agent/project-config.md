---
trigger: always_on
description: > **IMPORTANT**: `AGENTS.md` files are the source of truth for AI agent instructions. Always update the relevant `AGENTS.md` file when adding or modifying agent guidance. Do not add durable guidance to editor-specific rule files only.
---

# AGENTS.md - E-commerce Smart Agent

> **IMPORTANT**: `AGENTS.md` files are the source of truth for AI agent instructions. Always update the relevant `AGENTS.md` file when adding or modifying agent guidance. Do not add durable guidance to editor-specific rule files only.

## Maintenance Contract

- `AGENTS.md` is a living document.
- Keep this root file concise and router-like. Push narrow or conditional workflows into package-local `AGENTS.md` files.
- Update this file in the same PR when repo-level architecture, workflows, dependency boundaries, mandatory verification commands, or security processes materially change.
- For package-local material changes, update the nearest package `AGENTS.md` in the same PR.

## Read Order

1. Read this root `AGENTS.md` for repo-wide rules, commands, and routing.
2. Read the nearest nested `AGENTS.md` for the directory you are working in.
3. For architecture details, read [`docs/explanation/architecture/`](./docs/explanation/architecture/).
4. For project overview and screenshots, read [`README.md`](README.md).

## Context-Aware Loading

Use the right `AGENTS.md` for the area you're working in:

- **Agent implementations** (`@app/agents/**`) → [`app/agents/AGENTS.md`](app/agents/AGENTS.md)
- **LangGraph workflow** (`@app/graph/**`) → [`app/graph/AGENTS.md`](app/graph/AGENTS.md)
- **Intent recognition** (`@app/intent/**`) → [`app/intent/AGENTS.md`](app/intent/AGENTS.md)
- **Memory system** (`@app/memory/**`) → [`app/memory/AGENTS.md`](app/memory/AGENTS.md)
- **Tools** (`@app/tools/**`) → [`app/tools/AGENTS.md`](app/tools/AGENTS.md)
- **Retrieval** (`@app/retrieval/**`) → [`app/retrieval/AGENTS.md`](app/retrieval/AGENTS.md)
- **Evaluation** (`@app/evaluation/**`) → [`app/evaluation/AGENTS.md`](app/evaluation/AGENTS.md)
- **Observability** (`@app/observability/**`) → [`app/observability/AGENTS.md`](app/observability/AGENTS.md)
- **Tasks** (`@app/tasks/**`) → [`app/tasks/AGENTS.md`](app/tasks/AGENTS.md)
- **API layer** (`@app/api/**`) → [`app/api/AGENTS.md`](app/api/AGENTS.md)
- **Schemas** (`@app/schemas/**`) → [`app/schemas/AGENTS.md`](app/schemas/AGENTS.md)
- **Models** (`@app/models/**`) → [`app/models/AGENTS.md`](app/models/AGENTS.md)
- **Services** (`@app/services/**`) → [`app/services/AGENTS.md`](app/services/AGENTS.md)
- **Core** (`@app/core/**`) → [`app/core/AGENTS.md`](app/core/AGENTS.md)
- **Confidence** (`@app/confidence/**`) → [`app/confidence/AGENTS.md`](app/confidence/AGENTS.md)
- **Context** (`@app/context/**`) → [`app/context/AGENTS.md`](app/context/AGENTS.md)
- **Safety** (`@app/safety/**`) → [`app/safety/AGENTS.md`](app/safety/AGENTS.md)
- **WebSocket** (`@app/websocket/**`) → [`app/websocket/AGENTS.md`](app/websocket/AGENTS.md)
- **Utils** (`@app/utils/**`) → [`app/utils/AGENTS.md`](app/utils/AGENTS.md)
- **Tests** (`@tests/**`) → [`tests/AGENTS.md`](tests/AGENTS.md)
- **Admin frontend** (`@frontend/src/apps/admin/**`) → [`frontend/src/apps/admin/AGENTS.md`](frontend/src/apps/admin/AGENTS.md)
- **Customer frontend** (`@frontend/src/apps/customer/**`) → [`frontend/src/apps/customer/AGENTS.md`](frontend/src/apps/customer/AGENTS.md)

For any other area, this root file applies.

## Repo Map

- `@app/`: FastAPI backend, LangGraph workflow, agents, tools, services, observability, evaluation, memory, intent, retrieval, confidence, context, api, models, schemas, utils, websocket, tasks, core.
  - `@app/agents/`: Expert agent fleet (order, product, cart, payment, logistics, account, policy, complaint, supervisor, router, evaluator).
  - `@app/graph/`: LangGraph workflow compiler and runtime node layer.
    - `@app/graph/checkpointer.py` - OptimizedRedisCheckpoint with diff-based storage, compression, and TTL management.
    - `@app/graph/subgraphs.py` - Subgraph wrapper for agent state isolation.
  - `@app/intent/`: Intent recognition pipeline (classifier, multi-intent, safety, clarification, slot validation, topic switch).
    - `@app/intent/few_shot_loader.py` - Few-shot example loading for intent classification.
  - `@app/memory/`: Multi-tier memory system (structured PostgreSQL, vector Qdrant, fact extraction, summarization, compaction).
    - `@app/memory/structured_manager.py` - Structured memory manager for user profiles/preferences/facts.
  - `@app/tools/`: Tool layer for agents (product, cart, logistics, payment, account, complaint tools + registry).
  - `@app/tasks/`: Celery async tasks (memory, notifications, knowledge, refund, evaluation, continuous improvement, prompt effects, shadow testing).
    - `@app/tasks/alert_tasks.py` - Evaluate alert rules, check service health.
    - `@app/tasks/autoheal.py` - Self-healing orchestration module.
    - `@app/tasks/autoheal_tasks.py` - Restart stuck workers, clear expired Redis keys, check DB pool health.
    - `@app/tasks/checkpoint_tasks.py` - Cleanup old LangGraph checkpoints from Redis.
    - `@app/tasks/observability_tasks.py` - Post-chat async observability logging.
  - `@app/retrieval/`: Hybrid RAG retrieval (dense + sparse embeddings, reranker, query rewriter, Qdrant client).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mr-ZeLong/E-commerce-Smart-Agent](https://github.com/Mr-ZeLong/E-commerce-Smart-Agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
