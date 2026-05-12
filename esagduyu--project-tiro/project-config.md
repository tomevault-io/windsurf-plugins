---
trigger: always_on
description: Tiro is a local-first, open-source, model-agnostic reading OS for the AI age. It saves web pages and email newsletters as clean markdown, enriches them with AI-extracted metadata, and uses Claude Opus 4.6 for deep cross-document reasoning — daily digests, trust analysis, and learned reading preferences. Everything runs locally. The user owns their data.
---

# Project Tiro

## What This Is

Tiro is a local-first, open-source, model-agnostic reading OS for the AI age. It saves web pages and email newsletters as clean markdown, enriches them with AI-extracted metadata, and uses Claude Opus 4.6 for deep cross-document reasoning — daily digests, trust analysis, and learned reading preferences. Everything runs locally. The user owns their data.

Named after Cicero's freedman who preserved and organized his master's works for posterity. *"...without you the oracle was dumb." — Cicero to Tiro, 53 BC*

**Context:** Built solo for the "Built with Opus 4.6: Claude Code Hackathon" (Feb 10–16, 2026). Must be fully open source, built from scratch. See PROJECT_TIRO_SPEC.md for the full build plan.

## Architecture

```
Web UI (FastAPI serves HTML/JS at localhost:8000)
  ↕ REST API
FastAPI Backend (Python)
  ├── Ingestion Engine (readability-lxml + markdownify)
  ├── Intelligence Layer (Opus 4.6 — digests, analysis, preferences)
  ├── Lightweight Processing (Haiku — tags, entities, summaries)
  ├── Query Layer (ChromaDB semantic search + SQLite metadata)
  └── MCP Server (exposes knowledge base to Claude)
  ↕
Storage Layer (all local)
  ├── articles/*.md (markdown files with YAML frontmatter)
  ├── tiro.db (SQLite — metadata, preferences, stats)
  ├── chroma/ (ChromaDB — vector embeddings)
  └── config.yaml (user configuration)
```

**MCP server:** `tiro/mcp/server.py` exposes the library to Claude Desktop and Claude Code via 7 tools (see below).

## Tech Stack

- **Backend:** FastAPI, uvicorn, Python 3.11+
- **Content extraction:** readability-lxml, markdownify
- **Email parsing:** Python email stdlib → readability-lxml → markdownify
- **Storage:** SQLite (metadata), ChromaDB (vectors), markdown files on disk
- **Embeddings:** sentence-transformers (all-MiniLM-L6-v2) locally
- **AI (heavy):** Claude Opus 4.6 via Anthropic API (digests, analysis, preferences)
- **AI (light):** Claude Haiku 4.5 via Anthropic API (tags, entities, summaries)
- **Frontend:** Minimal HTML/CSS/JS served by FastAPI (Jinja2 templates)
- **MCP:** Python MCP SDK

## Key Conventions

- **Use `uv` for all Python version and dependency management** — never use pip directly. Use `uv pip install`, `uv venv`, `uv run`, etc. Dependencies defined in pyproject.toml.
- Python 3.11+, async throughout (async def for all route handlers)
- Use httpx for async HTTP calls
- Use python-frontmatter for reading/writing markdown with YAML frontmatter
- All files stored under a configurable `library_path` (default: `./tiro-library/`)
- Structured Anthropic API responses: always request JSON output, parse with error handling
- Logging via Python logging module, INFO level default
- Graceful error handling everywhere — never crash the server on bad input
- See PROJECT_TIRO_SPEC.md § "Data Models" for the full SQLite schema, markdown format, and ChromaDB collection spec
- See PROJECT_TIRO_SPEC.md § "Key Prompt Templates" for all Opus/Haiku prompt templates
- See PROJECT_TIRO_SPEC.md § "API Endpoints" for the full endpoint list

## Quick Start

```bash
uv sync                    # Install dependencies (creates venv automatically)
uv run tiro init           # Creates config.yaml, library, prompts for API key
uv run tiro run            # Start server on localhost:8000 (auto-opens browser)
uv run tiro run --lan      # Bind to 0.0.0.0 for LAN access (read on your phone)
```

Before starting the server, kill any existing process on port 8000:
```bash
lsof -ti :8000 | xargs kill -9
```

## API Endpoints

| Method | Path | Purpose |
|--------|------|---------|
| POST | /api/ingest/url | Save a web page |
| GET | /api/ingest/check?url=... | Check if URL is already saved (returns article data if found) |
| POST | /api/ingest/email | Save an uploaded .eml file |
| POST | /api/ingest/batch-email | Process all .eml files in a directory |
| GET | /api/articles | List articles (VIP pinned first) |
| GET | /api/articles/{id} | Get article with markdown content |
| PATCH | /api/articles/{id}/rate | Rate article (-1, 1, 2) |
| PATCH | /api/articles/{id}/read | Mark read, increment open count |
| GET | /api/sources | List sources with article counts |
| PATCH | /api/sources/{id}/vip | Toggle VIP status |
| GET | /api/articles/{id}/analysis | On-demand ingenuity/trust analysis |
| GET | /api/digest/today | Get/generate daily digest (all 3 variants) |
| GET | /api/digest/today/{type} | Get specific variant |
| GET | /api/search?q=... | Semantic search across articles |
| GET | /api/articles/{id}/related | Get related articles with connection notes |
| POST | /api/recompute-relations | Retroactively compute relations for all articles |
| POST | /api/classify | Classify unrated articles into tiers using Opus 4.6 |
| POST | /api/decay/recalculate | Recalculate content decay weights for all articles |
| GET | /api/stats?period=week\|month\|all | Reading stats (daily counts, top tags, top sources, streak) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [esagduyu/project-tiro](https://github.com/esagduyu/project-tiro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
