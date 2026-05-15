---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## CRITICAL RULES

### 1. Never Hardcode Configuration

**All ports, URLs, and connection strings MUST come from environment variables.**

```python
# ❌ NEVER
DATABASE_URL = "postgresql://localhost:5432/db"

# ✅ ALWAYS
DATABASE_URL = os.getenv("DATABASE_URL")  # Required - fail fast if missing
```

See `.env.example` for all required variables. Apps must fail fast with clear errors if env vars are missing.

### 2. Auto-Generated Files - Do Not Edit

These files are generated from `manifest.toml` via `airis init`:
- `package.json` (root)
- `pnpm-workspace.yaml`

**Edit `manifest.toml` instead, then run `airis init` to regenerate.**

### 3. Docker-First Development

Host package manager commands are forbidden. Use `airis` commands:

```bash
# ❌ FORBIDDEN on host
npm install / pnpm install / yarn install

# ✅ USE INSTEAD
airis up              # Start services
airis api-shell       # Enter container, then run commands inside
```

---

## Project Overview

**MindBase** is the memory substrate of the AIRIS ecosystem - a local-first AI conversation knowledge management system with semantic search via PostgreSQL + pgvector + Ollama.

**Key Integration**: Part of [AIRIS MCP Gateway](https://github.com/agiletec-inc/airis-mcp-gateway) which provides unified MCP access. MindBase exposes `mindbase_search` and `mindbase_store` tools.

**Stack**: Hybrid Python/TypeScript pnpm monorepo
- **Storage**: PostgreSQL 17 + pgvector
- **Embeddings**: OpenAI text-embedding-3-large (3072-dim, primary) → Ollama qwen3-embedding:8b (1024-dim, fallback)
- **API**: FastAPI (Python)
- **MCP Server**: TypeScript (stdio transport)

## Quick Reference

### Development Setup

```bash
cp .env.example .env          # Configure environment
airis up                      # Start PostgreSQL + API
airis model-pull              # Download embedding model (~4.7GB, first time only)
airis migrate                 # Run database migrations
airis health                  # Verify all services running
```

### Common Commands

| Command | Description |
|---------|-------------|
| `airis up` | Start all services |
| `airis down` | Stop all services |
| `airis restart` | Restart all services |
| `airis logs` | View all logs |
| `airis ps` | Show container status |
| `airis api-shell` | Enter API container for Python work |
| `airis db-shell` | Enter PostgreSQL shell |
| `airis test` | Run all tests |
| `airis health` | Check service health |
| `airis migrate` | Run database migrations |
| `airis model-pull` | Download embedding model |

### Service Endpoints (configured via .env)

- API: `http://localhost:${API_PORT}` (default: 18003)
- API Docs: `http://localhost:${API_PORT}/docs`
- PostgreSQL: `localhost:${POSTGRES_PORT}` (default: 15434)
- Ollama: `http://localhost:${OLLAMA_PORT}` (default: 11434)

## Codebase Structure

```
apps/
├── api/              # FastAPI backend (Python) - main entry: main.py
├── cli/              # CLI tool (TypeScript) - content generation pipeline
├── mcp-server/       # MCP Server (TypeScript) - main entry: index.ts
├── settings/         # Settings UI (React + Vite + Tailwind)
└── menubar-swift/    # Native macOS menubar app with auto-collection

libs/
├── collectors/       # Python conversation collectors (Claude, Cursor, ChatGPT, etc.)
├── generators/       # TypeScript article generators (Qiita, Zenn, Note publishing)
├── processors/       # TypeScript content processors
└── shared/           # Shared TypeScript types

supabase/migrations/  # PostgreSQL migrations (sequential SQL files)
tests/                # pytest tests (unit/integration/e2e markers)
```

### Key Entry Points

- **API**: `apps/api/main.py` → routes in `apps/api/api/routes/`
- **MCP Server**: `apps/mcp-server/index.ts` → tools in `tools/`, storage in `storage/`
- **CLI**: `apps/cli/index.ts` → uses `libs/generators/` for article generation
- **Collectors**: `libs/collectors/base_collector.py` (abstract) → source-specific implementations

## Architecture Notes

### Data Separation

```
~/Library/Application Support/mindbase/  # User data (NOT in git)
├── conversations/                        # Archived conversations by source
└── memories/                             # Markdown memories with frontmatter

~/github/mindbase/                        # Source code (Git-tracked)
```

### MCP Server Design

- **Transport**: stdio (for Claude Desktop/Windsurf/Cursor integration)
- **Storage backends**: Interface-based, swappable (PostgresStorageBackend, FileSystemMemoryBackend)
- **Error handling**: All tools return `{error: message}` on failure (no exceptions to client)
- **Hybrid memory**: Markdown files (human-readable) + PostgreSQL (semantic search)

### Embedding Strategy (Dual Provider)

- **Primary**: OpenAI `text-embedding-3-large` (3072-dim) — requires `OPENAI_API_KEY`
- **Fallback**: Ollama `qwen3-embedding:8b` (1024-dim) — local, free
- Both Python (`ollama_client.py`) and TypeScript (`storage/postgres.ts`) implement the same fallback logic
- Dimensions auto-detected; override with `EMBEDDING_DIMENSIONS`
- Index: ivfflat for pgvector similarity search

### Data Pipeline (Raw → Derived)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agiletec-inc/mindbase](https://github.com/agiletec-inc/mindbase) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
