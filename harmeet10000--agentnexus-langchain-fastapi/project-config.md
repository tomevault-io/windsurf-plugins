---
trigger: always_on
description: Prioritize deep, first principles thinking, insider-level knowledge that reveals how systems actually work beneath the abstraction layers. Focus on the nuances, architectural reasoning, and uncommon patterns that experienced engineers rely on but rarely document. Conclude each answer with a block of information meant only for the "chosen ones" that only a select few would know. It should contain insights that puts me one step ahead of everyone.
---


# Your role in this project 
Prioritize deep, first principles thinking, insider-level knowledge that reveals how systems actually work beneath the abstraction layers. Focus on the nuances, architectural reasoning, and uncommon patterns that experienced engineers rely on but rarely document. Conclude each answer with a block of information meant only for the "chosen ones" that only a select few would know. It should contain insights that puts me one step ahead of everyone. 

## Project Snapshot

- Project: `langchain-fastapi-production`
- Python: `3.12`
- Package manager: `uv`
- Formatter/linter: `ruff`
- Type checker: `ty`
- Framework stack: `FastAPI`, `Pydantic v2`, `LangChain`, `LangGraph`, `SQLAlchemy`, `Beanie`, `Redis`, `Celery`
- Architecture: modular monolith, feature-driven, async-first

## Project Structure

Use this structure when creating or moving code. Keep feature logic under `src/app/features`, reusable domain/runtime modules under `src/app/shared`, and cross-cutting helpers under `src/app/utils`.

```text
langchain-fastapi-production/
├─ .github/                          # Workflows, prompts, Copilot instructions
├─ caddy/                            # Caddy config
├─ docker/                           # Docker assets
├─ docs/                             # Documentation
├─ infra/                            # Cloud IaC (aws/azure/gcp)
├─ scripts/                          # Automation scripts
├─ src/
│  ├─ alembic/                       # Migrations
│  ├─ app/
│  │  ├─ api/                        # FastAPI routers (v1, etc.)
│  │  ├─ config/                     # Settings/configuration
│  │  ├─ connections/                # DB/Redis/other clients
│  │  ├─ examples/                   # Example code snippets and references
│  │  ├─ lifecycle/                  # Startup/shutdown and lifespan wiring
│  │  ├─ middleware/                 # HTTP/ASGI middleware and handlers
│  │  ├─ features/                   # Feature modules (auth, chat, crawler, ...)
│  │  ├─ shared/                     # Reusable app subsystems
│  │  │  ├─ agents/                  # Agent runtime building blocks
│  │  │  │  ├─ memory/               # Agent memory managers/integrations
│  │  │  │  ├─ orchestration/        # Agent routing/supervision logic
│  │  │  │  └─ tools/                # Agent tool implementations
│  │  │  ├─ crawler/                 # Shared crawling logic
│  │  │  ├─ document_processing/     # Parsing, chunking, ingestion helpers
│  │  │  ├─ langchain_layer/         # LangChain-specific adapters/components
│  │  │  ├─ langgraph_layer/         # LangGraph graphs/nodes/state
│  │  │  ├─ mcp/                     # MCP integrations and runtime
│  │  │  ├─ rag/                     # Retrieval and knowledge-layer modules
│  │  │  │  ├─ graphiti/             # Graphiti integrations
│  │  │  │  ├─ langextract/          # LangExtract integrations
│  │  │  │  ├─ multimodal/           # Multimodal RAG logic
│  │  │  │  └─ pageindex/            # PageIndex integrations
│  │  │  ├─ services/                # Shared service modules
│  │  │  └─ vectorstore/             # Shared vector store integrations
│  │  └─ utils/                      # Cross-cutting utilities (cache, messaging, ...)
│  ├─ database/
│  │  ├─ schemas/                    # Database schemas/models
│  │  └─ seeders/                    # Seed data
│  └─ tasks/                         # Background task entrypoints/jobs
└─ tests/
   ├─ unit/
   ├─ integration/
   ├─ e2e/
   └─ performance/
```

## Quality Gates

These tools are required for local development and CI. Keep this section aligned with `pyproject.toml`.

### Required commands

- `uv sync`
- `uv run ruff format src/`
- `uv run ruff check src/`
- `uv run ruff check --fix src/`
- `uv run ty check src/`

### Baseline lint and type expectations

- Use `ruff` as the source of truth for formatting and linting.
- Use `ty` as the source of truth for static typing.
- Use `uv` to run project tooling; do not suggest bare `ruff` or `ty` commands when `uv run ...` is available.
- **Treat `pyproject.toml` as the authoritative source for all enabled rules.** See `[tool.ruff.lint]` and `[tool.ty.rules]` sections.
- Before a PR or merge, run both `uv run ruff check src/` and `uv run ty check src/`.
- Do not weaken configured checks in examples, generated commands, CI snippets, or review advice unless the user explicitly asks for that change.
- When suggesting code, prefer patterns that satisfy the active async, security, import-order, and typing rules without needing ignores.


#### `ty` rule baseline (see [pyproject.toml](pyproject.toml#L388))


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Harmeet10000/AgentNexus-LangChain-FastAPI](https://github.com/Harmeet10000/AgentNexus-LangChain-FastAPI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
