---
trigger: always_on
description: Backend service for OpenArg — AI-powered analysis of Argentine government open data. Implements a pipeline that scrapes public data portals, generates vector embeddings, caches datasets, and answers natural-language queries using LLMs.
---

# OpenArg Backend

Backend service for OpenArg — AI-powered analysis of Argentine government open data. Implements a pipeline that scrapes public data portals, generates vector embeddings, caches datasets, and answers natural-language queries using LLMs.

## Stack

- **Framework:** FastAPI 0.115 + Uvicorn (async, UVLoop)
- **Database:** PostgreSQL 16 + pgvector (HNSW indexing, 1024-dim embeddings)
- **ORM:** SQLAlchemy 2.0 (async) + Alembic migrations
- **DI:** Dishka 1.6 (IoC container)
- **Workers:** Celery 5.4 + Redis 7 (broker + cache + results)
- **AI:** AWS Bedrock Claude Haiku 4.5 (primary LLM) + Google Gemini 2.5 Flash (fallback)
- **Embeddings:** AWS Bedrock Cohere Embed Multilingual v3 (1024-dim)
- **Pipeline:** LangGraph (stateful graph with checkpointing)
- **HTTP:** HTTPX (async client)
- **Auth:** PyJWT + bcrypt
- **Rate Limiting:** SlowAPI
- **Config:** TOML files + Pydantic settings
- **Logging:** structlog

## Architecture (Hexagonal / Ports & Adapters)

```
src/app/
├── domain/                                      # Domain layer
│   ├── entities/                                # Dataclass entities
│   │   ├── base.py                              # BaseEntity (id, created_at, updated_at)
│   │   ├── dataset.py                           # Dataset, DatasetChunk
│   │   ├── user_query.py                        # UserQuery, AgentTask
│   │   └── query_dataset_link.py
│   ├── ports/                                   # Abstract interfaces (ABC)
│   │   ├── source/data_source.py                # IDataSource: fetch_catalog, download_dataset
│   │   ├── dataset/dataset_repository.py        # IDatasetRepository: save, get_by_id, upsert
│   │   ├── llm/llm_provider.py                  # ILLMProvider, IEmbeddingProvider
│   │   ├── search/vector_search.py              # IVectorSearch: search_datasets, index_dataset
│   │   ├── sandbox/sql_sandbox.py               # ISQLSandbox: execute_readonly
│   │   └── cache/cache_port.py                  # ICacheService: get, set, delete
│   └── exceptions/                              # Domain exceptions
│
├── infrastructure/                              # Infrastructure layer
│   ├── adapters/
│   │   ├── source/
│   │   │   ├── datos_gob_ar_adapter.py          # IDataSource → datos.gob.ar CKAN
│   │   │   └── caba_adapter.py                  # IDataSource → CABA CKAN
│   │   ├── llm/
│   │   │   ├── bedrock_llm_adapter.py           # ILLMProvider → Claude Haiku 4.5 (AWS Bedrock, primary)
│   │   │   ├── bedrock_embedding_adapter.py     # IEmbeddingProvider → Cohere Embed Multilingual v3 (1024-dim)
│   │   │   ├── gemini_adapter.py                # ILLMProvider → Gemini 2.5 Flash (Google, fallback)
│   │   │   ├── anthropic_adapter.py             # ILLMProvider → Claude Sonnet 4 (Anthropic API)
│   │   ├── search/
│   │   │   └── pgvector_search_adapter.py       # IVectorSearch → pgvector
│   │   ├── sandbox/
│   │   │   └── pg_sandbox_adapter.py            # ISQLSandbox → read-only PG queries
│   │   ├── dataset/
│   │   │   └── dataset_repository_sqla.py       # IDatasetRepository → SQLAlchemy
│   │   └── cache/
│   │       └── redis_cache_adapter.py           # ICacheService → Redis
│   ├── resilience/                              # Fault tolerance
│   │   ├── retry.py                             # @with_retry decorator (exponential backoff + jitter)
│   │   └── circuit_breaker.py                   # In-memory circuit breaker (CLOSED→OPEN→HALF_OPEN)
│   ├── monitoring/                              # Observability
│   │   ├── health.py                            # HealthCheckService (postgres, redis, ddjj, sesiones)
│   │   ├── metrics.py                           # MetricsCollector singleton (requests, connectors, cache, tokens)
│   │   └── middleware.py                        # MetricsMiddleware (ASGI)
│   ├── persistence_sqla/
│   │   ├── mappings/                            # SQLAlchemy table ↔ entity mappings
│   │   ├── alembic/versions/                    # Migration files
│   │   └── provider.py                          # DB session provider
│   └── celery/
│       ├── app.py                               # Celery app config + task routing
│       └── tasks/
│           ├── scraper_tasks.py                 # scrape_catalog, index_dataset_embedding
│           ├── collector_tasks.py               # collect_dataset (download + cache in PG)
│           ├── embedding_tasks.py               # reindex_all_embeddings
│           └── analyst_tasks.py                 # analyze_query (plan → search → gather → analyze)
│
├── presentation/http/controllers/               # API layer
│   ├── root_router.py                           # Composes all routers under /api/v1
│   ├── health/health_router.py                  # GET /health, /health/ready
│   ├── datasets/datasets_router.py              # CRUD + scrape trigger
│   ├── query/query_router.py                    # Query submission + WebSocket stream
│   ├── query/smart_query_v2_router.py           # LangGraph pipeline (POST /smart + WS /ws/smart)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [colossus-lab/openarg_backend](https://github.com/colossus-lab/openarg_backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
