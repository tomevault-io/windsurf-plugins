---
trigger: always_on
description: **mnemory** is a self-hosted, two-tier memory system for AI agents, exposed as an MCP (Model Context Protocol) server over Streamable HTTP. It provides persistent memory with intelligent fact extraction, deduplication, and semantic search.
---

# AGENTS.md — Coding Agent Instructions for mnemory

## Project Overview

**mnemory** is a self-hosted, two-tier memory system for AI agents, exposed as an MCP (Model Context Protocol) server over Streamable HTTP. It provides persistent memory with intelligent fact extraction, deduplication, and semantic search.

- **Language**: Python 3.11+
- **Framework**: MCP SDK (`mcp.server.fastmcp.FastMCP`) with Starlette for HTTP
- **Core dependencies**: [qdrant-client](https://github.com/qdrant/qdrant-client) for vector storage, [openai](https://github.com/openai/openai-python) for LLM and embeddings
- **License**: Apache 2.0
- **Repository**: https://github.com/fpytloun/mnemory

## Architecture

```
mnemory/
├── server.py              # MCP server entry point, 16 tool definitions, health endpoint, auth middleware
├── config.py              # Configuration from environment variables (dataclasses)
├── categories.py          # Predefined category registry, validation, matching logic
├── memory.py              # Business logic layer (orchestrates vector + artifact stores)
├── llm.py                 # OpenAI-compatible LLM client with structured output support
├── embeddings.py          # OpenAI-compatible embedding client with batch support
├── prompts.py             # Unified extraction+classification+dedup prompt templates, query generation and rerank prompts for find_memories
├── ttl.py                 # TTL (Time-To-Live) utility functions for memory expiration
├── instructions.py        # Configurable MCP server instructions (passive/proactive/personality/managed modes)
├── session.py             # Server-side memory session tracking (MemorySession + SessionStore)
├── metrics.py             # Prometheus metrics collection, operation counters, Qdrant gauge aggregation
├── api/
│   ├── __init__.py        # FastAPI app factory, session store instance
│   ├── deps.py            # Auth + identity FastAPI dependency (reads contextvars)
│   ├── schemas.py         # Pydantic request/response models for all endpoints
│   ├── memories.py        # Memory CRUD + artifact + category REST endpoints
│   ├── recall.py          # POST /api/recall — combined initialize + search
│   ├── remember.py        # POST /api/remember — fire-and-forget memory storage
│   ├── sessions.py        # GET /api/sessions — session summary endpoints for UI
│   └── ui.py              # GET /api/whoami + GET /api/stats — management UI endpoints
├── ui/
│   ├── __init__.py        # Package marker
│   ├── tailwind.config.js # Tailwind CSS config with brand colors
│   ├── src/input.css      # Tailwind directives + component classes
│   └── static/            # Pre-built UI assets (HTML, JS modules: api/app/metrics/search/memories/sessions/graph, CSS, vendored libs)
└── storage/
    ├── vector.py          # Direct Qdrant vector store (insert, search, update, delete)
    ├── artifact.py        # Artifact store abstraction (S3 and filesystem backends)
    └── session.py         # Session persistence backends (SQLite, Redis, in-memory) + serialization
```

### Layer responsibilities

| Layer | File | Responsibility |
|---|---|---|
| **Transport** | `server.py` | MCP tool definitions, HTTP routing, auth, serialization |
| **REST API** | `api/` | FastAPI sub-app with OpenAPI spec, CRUD + intelligence endpoints |
| **Business Logic** | `memory.py` | Validation, reranking, core memory assembly, artifact lifecycle |
| **Categories** | `categories.py` | Category validation, prefix matching, counting |
| **TTL** | `ttl.py` | Expiration calculation, decay detection, reinforcement metadata |
| **Vector Storage** | `storage/vector.py` | Direct Qdrant client for all vector operations |
| **Artifact Storage** | `storage/artifact.py` | S3/MinIO and filesystem backends for binary artifacts |
| **Management UI** | `ui/`, `api/ui.py` | Built-in web UI — dashboard, search, browse/CRUD, graph (Alpine.js + Tailwind + Chart.js + D3.js) |
| **Instructions** | `instructions.py` | Configurable MCP server instructions (passive/proactive/personality modes) |
| **Sessions** | `session.py` | Server-side memory session tracking with write-through cache |
| **Session Storage** | `storage/session.py` | Session persistence backends (SQLite, Redis, in-memory) |
| **Metrics** | `metrics.py` | Prometheus metrics collection, operation counters, Qdrant gauge aggregation |
| **Configuration** | `config.py` | Environment variable parsing into dataclass configs |

### Key design decisions

1. **Direct Qdrant + OpenAI**: Uses Qdrant directly for all vector operations and OpenAI-compatible APIs for LLM and embeddings. A single unified LLM call handles fact extraction, per-fact classification, and deduplication simultaneously.

2. **Two-tier memory**: Fast memory (vector store, max 1000 chars, searchable) + slow memory (artifact store, up to 10MB, retrieved on demand). Artifacts are always attached to a parent fast memory.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fpytloun/mnemory](https://github.com/fpytloun/mnemory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
