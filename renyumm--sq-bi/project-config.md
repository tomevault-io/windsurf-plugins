---
trigger: always_on
description: **SQ-BI (智慧问数)** is a semantic-layer business intelligence platform for enterprise data (manufacturing, logistics, finance). Architecture uses LLM-as-scheduler + deterministic Skill-engine: the LLM parses natural language intents, extracts entities, and routes to pre-authored Skills (metrics/reports) that execute fixed SQL/API calls — never allowing LLM-generated SQL directly.
---

# Repository Guidelines

## Project Overview

**SQ-BI (智慧问数)** is a semantic-layer business intelligence platform for enterprise data (manufacturing, logistics, finance). Architecture uses LLM-as-scheduler + deterministic Skill-engine: the LLM parses natural language intents, extracts entities, and routes to pre-authored Skills (metrics/reports) that execute fixed SQL/API calls — never allowing LLM-generated SQL directly.

Product modules span data source management, metric dictionary, smart Q&A, report/Skill creation, PPT/Word export, secure sharing, and subscriptions. The repo is a uv-managed Python monorepo with a React frontend.

## Architecture & Data Flow

```
packages/contracts/ ─── sq-bi-contracts (Pydantic DTOs, enums, API envelope)
       │
       ▼
services/semantic/  ─── sq-bi-semantic (catalog, metric center, Skill CRUD)
       │
       ▼
services/runtime/   ─── sq-bi-runtime (LLM orchestration, SQL guardrails, query exec)
       │
       ├──▶ services/export/ ─── sq-bi-export (PDF/sharing/subscriptions)
       │
       └──▶ apps/web/        ─── React SPA (Vite + Tailwind)
```

- **LLM protocol**: `OpenAICompatClient` — sync httpx client, `chat(system, user)` returns `str`. Configurable via YAML + env override.
- **DB protocol**: `OracleExecutor` — connection pool via `oracledb`, thread-safe, TTL-cached schema description.
- **Service abstraction**: `@runtime_checkable` Protocol classes (`CatalogRepository`, `MetricRepository`, `SkillRepository`, `LLMProtocol`, `DBProtocol`).
- **API envelope**: `ApiResponse[T]` with exactly one of `data` or `error`. Every endpoint returns `request_id`.
- **Business logic errors**: custom exception classes mapped to `ErrorCode` enums and wrapped by handlers.
- **Config pattern**: YAML file + env var override (`_env_first()`), runtime settings persisted to `.local/runtime_settings.json`.
- **SQL guardrails**: `sqlglot` AST parsing, forbidden DDL/DML keywords, single-statement check, column validation against schema catalog, row-limit enforcement.

## Key Directories

| Path | Purpose |
|---|---|
| `packages/contracts/src/sq_bi_contracts/` | Shared domain types: `ids`, `enums`, `common`, `catalog`, `metrics`, `skills`, `query`, `reports`, `exports`, `api` |
| `services/semantic/src/sq_bi_semantic/` | Semantic catalog, metric center, Skill registry. `interfaces.py` (Protocols), `repository.py` (file-backed), `product_repository.py` (SQLite-backed, 1707 lines), `api.py` (30+ endpoints), `sql_validation.py`, `synonyms.py` |
| `services/runtime/src/sq_bi_runtime/` | Query runtime. `api.py` (1507 lines, 14 endpoints, report asset merging), `service.py` (AskDataService dataclass), `config.py`, `db.py` (OracleExecutor), `llm_client.py`, `guardrails.py`, `prompts.py` (6 system prompts), `schema_catalog.py`, `semantic_assets.py`, `settings.py`, `skill_loader.py` |
| `services/export/src/sq_bi_export/` | Export/sharing. `service.py` (ExportService, in-memory), `api.py` (12 routes), `renderers.py` (minimal PDF-1.4 generator) |
| `apps/web/src/` | React SPA. `App.tsx` (6392-line monolithic component), `api.ts` (typed fetch client), `index.css` (Tailwind v4 + theme tokens) |
| `.local/` | Runtime data: SQLite store, uploads, logs, runtime_settings.json |

## Development Commands

```bash
# Python workspace — sync all packages with dev extras
uv sync --all-packages --extra dev

# Run tests for a specific package
cd packages/contracts && uv run pytest
cd services/semantic && uv run pytest
cd services/runtime && uv run pytest
cd services/export && uv run pytest

# Start a service (dev)
cd services/runtime && uv run uvicorn sq_bi_runtime.api:app --reload

# Web frontend
cd apps/web && npm run dev      # dev server
cd apps/web && npm run build    # tsc -b && vite build (production)
cd apps/web && npm run lint     # eslint

# Demo
cd demo && streamlit run streamlit_app.py
cd demo && uv run uvicorn src.tms_askdata.api:app --reload

# Live smoke test
cd services/runtime && uv run python scripts/live_smoke.py
```

## Code Conventions & Common Patterns

### Python

- **File header**: every Python file starts with `from __future__ import annotations`.
- **Sync-only**: all service code is synchronous `def` — no async/await. Threading via `ThreadPoolExecutor` for parallel report asset execution.
- **Pydantic v2 models**: all DTOs use `model_config = ConfigDict(extra="forbid", populate_by_name=True)`. Contract models inherit `ContractModel` base.
- **API pattern**: `FastAPI` + `APIRouter(prefix="/api/v1")`, sync handlers. Every handler generates a `req_id = f"req_{uuid4().hex[:8]}"` and returns `ApiResponse(request_id=req_id, data=...)` or `_api_error(req_id, ErrorCode.VALIDATION_ERROR, str(exc))`.
- **Error handling**: business exceptions (e.g. `ExportNotFoundError`, `ExportPermissionError`) mapped to `ErrorCode` enums via `isinstance` checks in `_handle_error()`. catches `except Exception` with `# noqa: BLE001`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [renyumm/sq-bi](https://github.com/renyumm/sq-bi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
