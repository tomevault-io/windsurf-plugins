---
trigger: always_on
description: Cerefox is a user-owned knowledge memory layer for AI agents. It stores curated Markdown documents in Supabase (Postgres + pgvector), supports hybrid search (FTS + semantic), and exposes everything via MCP and REST so any AI agent can read and write.
---

# Cerefox - Project Guide

## What Is This

Cerefox is a user-owned knowledge memory layer for AI agents. It stores curated Markdown documents in Supabase (Postgres + pgvector), supports hybrid search (FTS + semantic), and exposes everything via MCP and REST so any AI agent can read and write.

Cerefox is **asynchronous shared memory, not a message bus**. It solves the persistent context problem: knowledge written in one context is findable in any other, dissolving boundaries between agents, sessions, human and machine, and across time. It does not handle real-time agent-to-agent communication; protocols like A2A handle that. Cerefox handles persistent memory.

Single-user, open-source (Apache 2.0), designed to be cheap/free to operate. See `docs/research/vision.md` for the full project vision.

## Tech Stack

- **Language**: Python 3.11+
- **Database**: PostgreSQL 16+ with pgvector (Supabase free tier or local Docker)
- **Embeddings**: OpenAI `text-embedding-3-small` (768-dim, cloud API); Fireworks AI as alternative; Edge Functions handle embedding server-side for agents
- **Web framework**: FastAPI (JSON API backend)
- **Web UI**: React + TypeScript SPA (Mantine UI, TanStack Query, Vite); served at `/app/`
- **CLI**: Click
- **Package management**: uv (pyproject.toml)
- **Testing**: pytest
- **Linting**: ruff

## Project Structure

```
cerefox/
├── CLAUDE.md                  # This file
├── pyproject.toml
├── docs/
│   ├── requirements-and-specs.md  # Source of truth for requirements
│   ├── solution-design.md         # Architecture and design decisions
│   ├── plan.md                    # Implementation plan with progress
│   └── TODO.md                    # Backlog and future ideas
├── src/
│   └── cerefox/
│       ├── __init__.py
│       ├── config.py              # Settings via pydantic-settings
│       ├── db/
│       │   ├── schema.sql         # Database schema
│       │   ├── rpcs.sql           # Search RPC functions
│       │   └── client.py          # Supabase/Postgres client wrapper
│       ├── chunking/
│       │   ├── markdown.py        # Heading-aware MD splitter
│       │   └── converters.py      # PDF/DOCX → MD (future)
│       ├── embeddings/
│       │   ├── base.py            # Embedder protocol/interface
│       │   └── cloud.py           # OpenAI/Fireworks REST API embedder
│       ├── ingestion/
│       │   └── pipeline.py        # Ingest documents → chunks → DB
│       ├── retrieval/
│       │   └── search.py          # Search + small-to-big assembly
│       ├── backup/
│       │   └── fs_backup.py       # File system / git backup
│       ├── api/
│       │   ├── app.py             # FastAPI application factory
│       │   ├── routes_api.py      # JSON API endpoints (/api/v1/)
│       │   └── deps.py            # Shared dependency injection
│       ├── mcp_server.py          # MCP stdio server (cerefox mcp)
│       └── cli.py                 # CLI entry point
├── frontend/                      # React + TypeScript SPA
│   ├── src/                       # Components, pages, hooks, API client
│   ├── vite.config.ts             # Vite build config (base: /app/)
│   └── package.json
├── web/
│   └── static/                    # Static assets (logo, favicon)
├── scripts/
│   ├── db_deploy.py           # Deploy schema to Supabase/Postgres
│   ├── db_migrate.py          # Apply schema migrations
│   ├── backup_create.py       # Take a local backup of the knowledge base
│   └── backup_restore.py      # Restore from a backup
├── tests/
│   ├── chunking/
│   ├── embeddings/
│   ├── ingestion/
│   ├── retrieval/
│   └── conftest.py
├── docker-compose.yml
└── Dockerfile
```

## Development Conventions

### Code Style
- Use ruff for linting and formatting (line length 100)
- Type hints on all public functions
- Docstrings only where the purpose isn't obvious from the name/signature
- Prefer simple, flat code over abstractions — don't create a helper for something used once

### Naming
- Database tables: `cerefox_` prefix (e.g., `cerefox_documents`, `cerefox_chunks`)
- Database RPCs: `cerefox_` prefix (e.g., `cerefox_hybrid_search`)
- Python modules: snake_case, short names
- Config: environment variables with `CEREFOX_` prefix

### Architecture Principles
- **Pluggable embedders**: all embedders implement the `Embedder` protocol (see `embeddings/base.py`)
- **Markdown-first**: all content is converted to markdown before chunking/storage
- **Fire-and-forget ingestion**: ingestion can be async; failures log errors but don't block
- **Parameterized limits**: response size limits, chunk sizes, etc. are configurable via settings
- **Two-table design**: `cerefox_documents` (document-level) + `cerefox_chunks` (chunk-level) for clean separation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fstamatelopoulos/cerefox](https://github.com/fstamatelopoulos/cerefox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
