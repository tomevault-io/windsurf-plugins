---
trigger: always_on
description: > **WARNING — PARTIALLY OUTDATED (March 2026)**
---

# Airweave Search Rules

> **WARNING — PARTIALLY OUTDATED (March 2026)**
>
> This file documents the **legacy V1 search module** (`airweave/search/`), which
> uses an operation-based pipeline with Qdrant, Cerebras/OpenAI/Groq providers,
> and structured-output LLM calls.
>
> The **new V2 search** lives in `airweave/domains/search/` and implements a
> three-tier architecture:
>
> - **Instant** — direct vector search, no LLM
> - **Classic** — vector search + reranking + answer generation (replaces V1)
> - **Agentic** — multi-turn tool-calling agent loop with search/count/navigate/read/collect/finish tools
>
> Key differences from V1:
> - **Vector DB**: Vespa (not Qdrant)
> - **LLM adapters**: `airweave/adapters/llm/` with Together AI + Anthropic fallback (not Cerebras/Groq)
> - **Reranker**: `airweave/adapters/reranker/` with Cohere
> - **Filters**: `domains/search/types/filters.py` — FilterableField/FilterOperator/FilterGroup (not Qdrant native filters)
> - **Streaming**: SSE events defined in `domains/search/agentic/events.py` (tool_call, thinking, search_results, etc.)
> - **API**: `POST /collections/{id}/search/{tier}` where tier is `instant`, `classic`, or `agentic`
> - **Config**: `domains/search/config.py` — model specs, token budgets, tier defaults
>
> **When working on `domains/search/`**, prefer reading the actual code over this
> file. The sections below remain accurate for the legacy `airweave/search/` module only.

## Overview (Legacy V1)

The search module (`@search/`) implements a **modular, pipeline-based architecture** with composable operations.  It aims to maintain search quality and flexibility.

## Core Architecture

### Operation-Based Pipeline

```python
SearchRequest → SearchFactory → SearchContext → SearchOrchestrator → SearchResponse
                                    ↓
                            [Operations Pipeline]
```

Each operation:
- Implements `SearchOperation` abstract base class
- Declares dependencies explicitly
- Reads/writes to shared state dictionary
- Can be optional (graceful failure)
- Executes asynchronously

### Request Flow
1. **Endpoint** (`api/v1/endpoints/search.py`) → Creates/receives `SearchRequest`
2. **SearchService.search()** → Main entry point
3. **SearchFactory.build()** → Creates `SearchContext` with enabled operations
4. **SearchOrchestrator.run()** → Executes operations in dependency order
5. **Operations** → Execute in topologically sorted order
6. **Qdrant destination** → Vector search execution
7. **Data Persistence** → Save search query to `search_queries` table via `CRUDSearchQuery`

### API Endpoints

Search endpoints are defined in `api/v1/endpoints/search.py` and mounted under `/collections` prefix in `api/v1/api.py`:

```python
# In api/v1/api.py
from airweave.api.v1.endpoints import search
api_router.include_router(search.router, prefix="/collections", tags=["collections"])
```

**Available Endpoints:**

1. **GET `/collections/{readable_id}/search`** - Legacy endpoint (DEPRECATED)
   - Maintained for backwards compatibility
   - Query parameters: `query`, `response_type`, `limit`, `offset`, `recency_bias`
   - Returns `LegacySearchResponse` with `status` field
   - Automatically converts to new format internally
   - Adds deprecation headers

2. **POST `/collections/{readable_id}/search`** - Main search endpoint (RECOMMENDED)
   - Accepts both `SearchRequest` (new) and `LegacySearchRequest` (old) schemas
   - Supports Qdrant native filters via `filter` field
   - Full control over all search features
   - Returns `SearchResponse` (new) or `LegacySearchResponse` (legacy) based on input schema
   - Adds deprecation headers for legacy requests

3. **POST `/collections/{readable_id}/search/stream`** - Streaming search with SSE
   - Accepts both `SearchRequest` and `LegacySearchRequest`
   - Returns Server-Sent Events (SSE) stream
   - Real-time progress updates via Redis pubsub
   - Automatically converts legacy requests

4. **GET `/collections/internal/filter-schema`** - Filter schema endpoint
   - Returns Qdrant Filter JSON schema for frontend validation
   - Public endpoint for building UI filter builders

All endpoints use the same underlying `SearchService.search()`, ensuring consistent behavior and quality.

### Input/Output Schemas

**SearchRequest** (new schema - `schemas/search.py`):
```python
query: str                                   # Search text (required)
retrieval_strategy: Optional[RetrievalStrategy]  # "hybrid", "neural", or "keyword"
filter: Optional[QdrantFilter]               # Qdrant native filter object
offset: Optional[int]                        # Pagination offset
limit: Optional[int]                         # Results per page
temporal_relevance: Optional[float]          # Recency weight (0-1, default: 0.3)
expand_query: Optional[bool]                 # Generate query variations
interpret_filters: Optional[bool]            # Extract filters from natural language
rerank: Optional[bool]                       # LLM-based reranking
generate_answer: Optional[bool]              # AI-generated completion
```

**SearchResponse** (new schema - `schemas/search.py`):
```python
results: List[Dict]                          # Search results
completion: Optional[str]                    # AI-generated answer (if generate_answer=True)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [airweave-ai/airweave](https://github.com/airweave-ai/airweave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
