---
trigger: always_on
description: This repository contains the backend for the Fagoon AI Agents Workflow platform, built with FastAPI. It handles complex AI orchestration, including RAG-enabled chat, background video generation, image generation, text-to-speech, tool-calling execution, and deep integration with Google Workspace.
---

# Fagoon AI Agents Workflow

This repository contains the backend for the Fagoon AI Agents Workflow platform, built with FastAPI. It handles complex AI orchestration, including RAG-enabled chat, background video generation, image generation, text-to-speech, tool-calling execution, and deep integration with Google Workspace.

---

## Mandatory Rule for AI Agents
**CRITICAL:** Any major architectural, structural, or infrastructural changes made to this project MUST be documented by updating this `GEMINI.md` file immediately. This ensures all future agents understand the current system state. Do not proceed with major architecture refactors without updating this document.

---

## Tech Stack & Architecture

- **Framework:** FastAPI (Python 3.12+)
- **Package Management:** `uv` with `pyproject.toml`
- **Primary Database:** PostgreSQL (accessed via SQLAlchemy `AsyncSession` & `asyncpg`)
- **Vector Database:** PostgreSQL via `pgvector` (for RAG document chunks)
- **Migrations:** Alembic
- **Background Jobs / Queues:** Celery (with Redis broker, Eventlet execution pool) or Inline Async Queue (Lite Mode).
- **Real-time Communication:** Server-Sent Events (SSE) for streaming LLM responses, WebSockets for video generation status.
- **LLM/AI Integrations:** OpenAI, Anthropic, Google GenAI (Gemini/Veo), Groq, Hugging Face, Fal AI, ElevenLabs, Kokoro.
- **Architecture Pattern:** Layered/Modular (Routers -> Services -> Providers -> Data/Storage).

---

## Dual-Mode Runtime Architecture (Lite & Full)

Fagoon supports two modes of operation:
1. **Lite Mode** (Default/Single-user local package):
   - **Launch Command:** `fagoon up` or with local fallbacks (`fagoon up --ollama`).
   - **Infrastructure:** Runs entirely self-contained without heavy external dependencies. It uses:
     - **In-memory Limiter:** `MemoryRateLimiter`
     - **In-memory Cache:** `MemoryCache`
     - **Inline Async Task Queue:** `InlineTaskQueueWithSyncSupport`
     - Single-process concurrency model (`WEB_CONCURRENCY=1` enforced at startup).
     - Secrets (such as `JWT_SECRET` and `encryption_key`) are auto-generated on first boot and persisted to `<DATA_DIR>/config.json`.
   - **No Redis/Celery imports:** Business logic modules must never import `redis` or `celery` directly. They route all requests through `app.state.limiter`, `app.state.queue`, and `app.state.cache`. This restriction is validated by `import-linter`.
2. **Full Mode** (Production multi-user environment):
   - **Launch Command:** `docker compose -f deploy/docker-compose.full.yml up --build` or setting environment variable `LITE_MODE=false`.
   - **Infrastructure:** Employs Redis as a broker and Celery with an Eventlet execution pool for distributed, multi-worker asynchronous background processing (such as long-running video generation).

Mode selection is governed by the `LITE_MODE` environment variable, resolved at startup in `src/core/runtime.py`.

---

## Strict Layer & Module Rules

- **Routers (`src/api/v1/routers/`):** ONLY routing and request validation (Pydantic). Dependency injection happens here. No business logic. Subdirectories categorize domains (`agents`, `authentication`, `external`, `misc`, `upgrade`, `video_gen`, `workflow`, `workspace`).
- **Services (`src/services/`):** Core business logic. Orchestrates calls to models, storage, and external providers.
    - `tool_handlers/`: Specific execution logic for AI tools (e.g., `web_search_handler.py`, `mermaid_handler.py`, `deep_research_handler.py`).
    - `rag/`: Document ingestion and chunking (`ingestion_manager.py`).
    - `google_workspace/`: Logic for Docs, Drive, and Gmail APIs.
    - `channel_adapter/`: Webhook processing for external messaging platforms.
    - `nosql/`: Database wrapper services (Legacy naming, but currently houses `postgres_services.py`).
- **Models & Schemas:** 
    - `src/models/sql/`: SQLAlchemy Base models (`models.py`) defining the exact PostgreSQL schema.
    - `src/schemas/`: Pydantic schemas for request/response validation. Keep them strictly typed.
- **Storages (`src/storages/`):** Abstractions for `file_storage` and vector storage (`vectordb_storages/pgvector.py`).
- **LLMs & Providers (`src/llms/`, `src/providers/`):** Wrappers for external AI API clients.
- **Core (`src/core/`):** Application globals, settings (`settings.py`), database connection pool (`PostgresManager`), exception handlers.

---

## Lifespan Singletons
Heavy clients are initialized once during FastAPI lifespan (`src/launch_server.py`) and attached to `app.state`. Do NOT instantiate these per request:
- `app.state.postgres_manager`: Connection pool for PostgreSQL.
- `app.state.httpx_client`: Global HTTP client.
- `app.state.vector_store`: Singleton `PgVectorStorage`.
- `app.state.agent_manager` & `app.state.chat_orchestrator`: Core orchestration singletons.

---

## Core Request Lifecycles & Component Flows

### 1. RAG-Enabled Agent Chat (Server-Sent Events)
The system uses Retrieval-Augmented Generation (RAG) to provide AI agents with specific document context.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Fagoon-AI/upgrade](https://github.com/Fagoon-AI/upgrade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
