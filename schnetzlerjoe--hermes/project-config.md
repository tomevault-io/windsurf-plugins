---
trigger: always_on
description: provides lazy access.
---

# CLAUDE.md — Agent Instructions for hermes-financial

This file provides context for AI coding assistants working on this codebase.

## Project Overview

**hermes-financial** is an open-source Python library (MIT) providing a
multi-agent financial research framework built on LlamaIndex. It ships with
data retrieval tools, document ingestion pipelines, output generation (Excel,
Word/PDF, charts), and composable agents for equity research.

A separate closed-source chatbot product (hermes-app) is built on top of this
library. This repo contains only the open-source library.

## Tech Stack

- **Python >= 3.10** — modern type hints (`X | None`, not `Optional[X]`)
- **LlamaIndex** — agent framework, tool wrappers, RAG pipelines
- **Pydantic v2** — configuration and data models
- **edgartools** — high-level SEC EDGAR API: XBRL parsing, structured financial statements, filing sections; sync-only, all calls wrapped in `asyncio.to_thread()`
- **httpx** — async HTTP client for FRED, Yahoo Finance, and SEC EFTS full-text search
- **openpyxl** — Excel workbook creation and manipulation
- **python-docx** — Word document generation
- **matplotlib** — chart generation (static PNG)
- **ChromaDB** — local vector store for document retrieval
- **BeautifulSoup4 / lxml** — used by `ingestion/sec_parser.py` for filing HTML → LlamaIndex nodes; not used by the SEC EDGAR tools

## Package Manager

We use **uv** as the primary package manager. All commands should use `uv run`.
The project must also remain pip-installable via `pyproject.toml`.

```bash
uv sync --all-extras    # install everything
uv run pytest           # run tests
uv run ruff check       # lint
uv run mypy hermes/     # type check
```

## Architecture

### Module Layout

```
hermes/
├── config.py        — Pydantic config with HERMES_ env prefix
├── registry.py      — Tool/agent registry (no global state)
├── core.py          — Hermes facade class
├── tools/           — Data + output tools (each exposes create_tools())
├── agents/          — Specialist agents (each inherits HermesAgent)
├── ingestion/       — Document parsing and ChromaDB indexing
└── infra/           — Cache, rate limiter, streaming events
```

### Key Patterns

- **Tool modules** expose a `create_tools() -> list[FunctionTool]` factory.
  Each tool function is a standalone async function that can be used without
  the agent layer.
- **Agent classes** inherit from `HermesAgent` (ABC) and implement `get_tools()`.
  The `agent_type` attribute selects between `FunctionAgent` (tool-calling)
  and `ReActAgent` (chain-of-thought).
- **Registry** — tools and agents are registered by name with optional tags.
  The `Hermes` class owns a `Registry` instance (no singletons).
- **Config** — `HermesConfig` is a Pydantic model with env var support.
  `configure()` creates/updates a module-level instance; `get_config()`
  provides lazy access.
- **Rate limiting** — `infra/rate_limiter.py` provides token bucket limiters.
  edgartools manages its own 10 req/sec rate limit for SEC EDGAR internally.
  Our limiter is used for FRED, Yahoo Finance, and SEC EFTS (full-text search).
- **Caching** — `infra/cache.py` provides a file-based cache with per-item TTL.
  The `cached_request()` helper in `tools/_base.py` wraps the fetch-or-cache
  pattern. edgartools manages its own HTTP cache for SEC filings internally;
  our cache is used for EFTS search results and other non-edgartools calls.
- **edgartools identity** — `hermes/tools/sec_edgar.py` calls `edgar.set_identity()`
  once on first use, pulling `sec_user_agent` from `HermesConfig`. All edgartools
  calls are sync and must be wrapped in `asyncio.to_thread()`.

### Agent Types

| Agent | Type | Rationale |
|-------|------|-----------|
| SecFilingsAgent, MacroAgent, MarketDataAgent, NewsAgent | FunctionAgent | Straightforward data retrieval |
| ModelingAgent, ReportAgent, ResearchOrchestrator | ReActAgent | Multi-step reasoning required |

Default to `FunctionAgent`. Only use `ReActAgent` where the reasoning chain helps.

## Conventions

### Code Style

- **ruff** for linting and formatting (line-length 99, target py310)
- **Type hints everywhere** — use `X | None` not `Optional[X]`
- **`from __future__ import annotations`** at the top of every module
- **Docstrings** — use Google/NumPy style, include Args/Returns sections
- **Logging** — `logger = logging.getLogger(__name__)` per module
- **No global mutable state** except the config singleton and rate limiter pool
- Async by default for all network-facing code

### Naming

- Tool functions: `verb_noun` (e.g., `get_company_facts`, `excel_write_cells`)
- Agent classes: `{Domain}Agent` (e.g., `SecFilingsAgent`, `MacroAgent`)
- Config fields: `snake_case`, env vars are `HERMES_UPPER_SNAKE`
- Cache namespaces: `source_type` (e.g., `sec_facts`, `fred_series`)

### Testing

- Tests in `tests/` mirroring the package structure
- Use `pytest` with `pytest-asyncio` (mode=auto)
- Mock HTTP calls for unit tests; mark live network tests with `@pytest.mark.network`
- For SEC EDGAR tests: mock edgartools objects (`edgar.Company`, `edgar.get_by_accession_number`)
  rather than HTTP calls — patch at the `edgar.*` level, not at the HTTP client level;
  patch `hermes.tools.sec_edgar._ensure_identity` to skip identity setup in tool tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [schnetzlerjoe/hermes](https://github.com/schnetzlerjoe/hermes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
