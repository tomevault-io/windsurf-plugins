---
trigger: always_on
description: Guide for AI assistants working in `ai-study-hub-rag-service`. Everything below is grounded in the current source tree.
---

# Repository Guidelines

Guide for AI assistants working in `ai-study-hub-rag-service`. Everything below is grounded in the current source tree.

## Project Overview

**AI Study Hub RAG Service** — a FastAPI / Python service that powers the AI features of the AI Study Hub platform. It owns document **ingestion** (download → parse → parent-child chunk → embed → index), **hybrid retrieval** (BM25 + pgvector dense + Jina cross-encoder re-rank), and **RAG generation** (Gemini) with numeric `[N]` citations. It stores vectors in PostgreSQL + pgvector and shares one `aistudyhub` database (plus one `INTERNAL_API_SECRET`) with the sibling **Java backend** (`ai-study-hub-api`). Runs on uvicorn, port `8000`.

## Architecture & Data Flow

Single FastAPI app (`main.py`) exposing REST endpoints. There is no DB ORM layer — persistence is raw `psycopg2` over a process-wide `ThreadedConnectionPool` (`app/database/pool.py`). External LLM/embedding/rerank clients are process-wide singletons (`app/core/clients.py`), warmed at startup.

```
Java backend ──HTTP──▶ FastAPI (main.py)
                          │
   ingest endpoints       │           chat endpoint: POST /api/v1/chat
   POST /rag/extract      │           │
   POST /rag/index        │           ├─ guardrail (runs BEFORE router, in main.chat_router):
   PATCH /rag/.../visibility│         │     • validate_input + detect_prompt_injection (always ON)
   DELETE /rag/documents/{id}│        │     • check_policy_topic (LLM) only if ENABLE_POLICY_GUARDRAIL=1
                          │           │     • block ─▶ HTTP 200 canned refusal (no retrieval/generation)
                          │           ▼
                          └─▶ route_chat_request (deterministic: SMALLTALK → SUMMARY → QA)
                                        QA branch: retrieve_documents()
                                        ├─ BM25 (parent docs, filtered by document_id)
                                        ├─ dense pgvector cosine (HNSW, k=25)
                                        ├─ EnsembleRetriever (BM25 0.3 / dense 0.7)
                                        ├─ Jina re-rank → top context
                                        └─ Gemini generation (+ history) → [N] citations
   ◀── callback (X-Internal-Secret) ─ send_callback() ──▶ backend
```

**Observability.** Every `/chat`, `/quiz/generate`, `/flashcard/generate`, and ingestion BackgroundTask opens a **Langfuse** root trace — `trace_chat` / `trace_material` / `trace_ingest` in `app/core/langfuse_client.py` — with sub-stages wrapped in `lf_span(name)` and LLM calls auto-captured via LangChain callbacks. All tracing is **fail-open at runtime** (`LANGFUSE_ENABLED=0`, missing keys, or any SDK error → no-op); the request path is never affected. See *Code Conventions → Instrumentation* for the helper inventory + per-route metadata.

**Ingestion is two-phase** (`app/services/ingestion.py`):
- `_extract`: download presigned file → load by extension → enrich metadata (page/chunk citations + `document_id`) → parent-child chunk via `ParentDocumentRetriever._split_docs_for_adding` → store parent docs in `LocalFileStore` (`parent_docs_store/`) → insert child chunks into `document_chunks` with **`embedding = NULL`**.
- `_index`: `embed_pending_chunks(document_id)` — embed all `embedding IS NULL` chunks (one `embed_documents` call, 1536-dim Gemini) + per-row `UPDATE` → `update_bm25()`. Idempotent.
- `process_document_task` (PRIVATE docs) = `_extract` + `_index` + summary, then callback `SUCCESS`. `extract_document_task` (PUBLIC docs) = `_extract` only + summary, callback `EXTRACTED`. `index_document_task` (after approval) = `_index`, callback `SUCCESS`.

Callbacks (`send_callback`) POST to `BACKEND_CALLBACK_URL` (`/api/v1/internal/documents/callback`) with header `X-Internal-Secret`, body `{document_id, status, summary}`, retried 3× with exponential backoff.

## Key Directories

```
main.py                     FastAPI app: endpoints, request models, startup warmup
app/
├── core/
│   ├── config.py           Settings (DATABASE_URL, BACKEND_CALLBACK_URL, INTERNAL_API_SECRET, LANGFUSE_*, ...)
│   ├── clients.py          Singleton Gemini LLM + Jina reranker (process-wide)
│   ├── performance.py      Instrumentation: start_trace() / stage() / trace.emit() → logs/performance.log
│   └── langfuse_client.py  Langfuse SDK v4 singleton + fail-open helpers: get_langfuse / get_langchain_callbacks
│                           / lf_span(name) / trace_chat / trace_material / trace_ingest
├── database/
│   ├── pool.py             ThreadedConnectionPool + db_connection() (rollback on exit, test-on-borrow SELECT 1, TCP keepalives)
│   ├── vector_store.py     Custom PostgresVectorStore over pgvector (add_texts, embed_pending_chunks,
│   │                       delete_by_document_id, update_chunk_visibility, similarity_search_by_vector)
│   └── document_store.py   document lookups (summary, title, user document ids)
├── services/
│   ├── ingestion.py        _extract / _index / process / extract / index tasks + delete + visibility
│   ├── retrieval.py        Hybrid retrieval (BM25+dense) + Jina re-rank; optional follow-up query-rewrite

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SWP-SUM26-AI-STUDY-HUB/ai-study-hub-rag-service](https://github.com/SWP-SUM26-AI-STUDY-HUB/ai-study-hub-rag-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
