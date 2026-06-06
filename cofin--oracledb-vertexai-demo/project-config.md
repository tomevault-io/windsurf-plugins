---
trigger: always_on
description: This file gives agents a current project context for the Cymbal Coffee demo.
---

# Oracle 26ai + Vertex AI Demo - Agent Configuration

This file gives agents a current project context for the Cymbal Coffee demo.
Keep it short, code-grounded, and aligned with `.agents/index.md`.

## Project Overview

**Purpose**: Coffee recommendation demo using Oracle 26ai vector search,
Google Vertex AI embeddings, Google ADK conversation orchestration, and
store-aware chat planning for locations, inventory, and maps.

**Current stack**:

- **Framework**: Litestar 2 with HTMX, Jinja templates, and litestar-vite template mode.
- **Server**: Granian via `uv run coffee run`.
- **Database**: Oracle 26ai with SQLSpec, named SQL files, JSON, BOOLEAN, and `VECTOR(3072, FLOAT32)`.
- **AI**: Vertex AI `gemini-embedding-001` embeddings and Gemini Flash-Lite chat/intent calls.
- **Agent runtime**: Google ADK 2 Workflow/BaseNode runner with Oracle-backed ADK sessions.
- **DI**: Dishka with three app providers in `src/app/ioc.py`.
- **Planned components**: store coordinates/inventory, deterministic store and
  product-availability chat routes, browser location opt-in, no-key Maps URLs,
  optional Maps Embed, and settings contract cleanup.

## Project Context

The active agent context lives under `.agents/`:

- [Project context index](.agents/index.md)
- [Workflow](.agents/workflow.md)
- [Patterns](.agents/patterns.md)
- [Project knowledge guide](.agents/knowledge/project-guide.md)
- [Architecture guide](.agents/knowledge/guides/architecture.md)
- [Oracle vector search guide](.agents/knowledge/guides/oracle-vector-search.md)
- [ADK agent guide](.agents/knowledge/guides/adk-agent-patterns.md)

Durable lessons must live in `.agents/knowledge/`, `.agents/patterns.md`, or
`.agents/workflow.md`. `.agents/archive/` is ignored disposable history; do not
link readers there as required context.

## Development Commands

```bash
# Setup
make install
uv run python manage.py init --run-install

# Local Oracle
make start-infra
uv run coffee upgrade

# App
uv run coffee run
uv run coffee bulk-embed
uv run coffee export-fixtures
uv run coffee clear-cache
uv run coffee model-info

# Verification
make lint
make test
make coverage
```

`coffee` is the hand-rolled app CLI. `coffee upgrade` is the packaged/end-user
install command; it applies migrations and loads committed fixtures. Keep raw
SQLSpec developer commands such as downgrade/current on `python manage.py
database ...`, not on `coffee`. Keep `bulk-embed` and `export-fixtures` on
`coffee`; they are maintainer lifecycle commands for committed demo data. Keep
large command workflows under `app.cli._helpers`; keep small command-local
helpers in `commands.py`. Do not add compatibility shim or facade modules. Use
`@async_inject` for async Click commands.

## Project Structure

```text
src/app/
├── cli/
│   ├── _helpers/          # substantial private CLI workflow helpers
│   ├── commands.py        # click command declarations
│   ├── main.py            # coffee click group
│   └── utils.py           # async_inject
├── db/
│   ├── fixtures/          # committed demo fixture data
│   ├── migrations/        # SQLSpec migrations
│   └── sql/               # named SQL files
├── domain/
│   ├── chat/
│   │   ├── controllers/
│   │   ├── schemas/
│   │   └── services/
│   ├── products/
│   │   ├── controllers/
│   │   ├── schemas/
│   │   └── services/
│   ├── system/
│   │   ├── controllers/
│   │   ├── schemas/
│   │   └── services/
│   └── web/
│       ├── controllers/
│       ├── static/
│       └── templates/
├── lib/                   # settings, DI, logging, SQLSpec service exports
├── server/                # Litestar app factory and plugin wiring
└── utils/                 # shared helpers

src/tests/
├── api/
├── integration/
└── unit/

tools/
├── cli/
└── oracle/
```

## Core Patterns

### SQLSpec Services

Use named SQL from `db/sql/*.sql` and typed result mapping. Do not reintroduce
manual vector packing; SQLSpec Oracle handles Python `list[float]` values.

```python
from app.config import db_manager
from app.domain.products.schemas import ProductMatch
from app.lib.service import SQLSpecAsyncService


class ProductService(SQLSpecAsyncService[OracleAsyncDriver]):
    async def search_by_vector(self, query_embedding: list[float]) -> list[ProductMatch]:
        return await self.driver.select(
            db_manager.get_sql("vector-search-products"),
            query_vector=query_embedding,
            threshold=0.5,
            limit=5,
            schema_type=ProductMatch,
        )
```

### Dishka DI

Use handler-argument injection. `setup_dishka` and `DomainPlugin(use_dishka_router=True)`
resolve dependencies; route-level `@inject` decorators are not the local pattern.

```python
from app.domain.products.services import ProductService
from app.lib.di import Inject


async def list_products(products_service: Inject[ProductService]) -> ProductList:
    products, total = await products_service.list_with_count()
    return ProductList(items=products, total=total)
```

`src/app/ioc.py` must not use `from __future__ import annotations`; Dishka reads
provider annotations at runtime.

### Vertex AI

Use `task_type="RETRIEVAL_QUERY"` for user search queries and
`task_type="RETRIEVAL_DOCUMENT"` for product/document embeddings. Runtime

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cofin/oracledb-vertexai-demo](https://github.com/cofin/oracledb-vertexai-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
