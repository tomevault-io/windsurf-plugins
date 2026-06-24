---
trigger: always_on
description: This file provides shared guidance for coding agents working in this repository.
---

# AGENTS.md

This file provides shared guidance for coding agents working in this repository.

## Project Overview

**OrionBelt Semantic Layer** is a SaaS semantic layer engine that compiles and executes YAML semantic models (OBML format) as analytical SQL across 8 database dialects: BigQuery, ClickHouse, Databricks, Dremio, DuckDB, MySQL, Postgres, Snowflake. It exposes all capabilities through a REST API (FastAPI). An MCP server is available as a separate thin client in [orionbelt-semantic-layer-mcp](https://github.com/ralforion/orionbelt-semantic-layer-mcp).

## Commands

```bash
uv sync                           # all deps (dev, docs, ui, flight, drivers)

uv run orionbelt-api              # REST API on :8000
uv run orionbelt-ui               # Gradio UI

uv run pytest                     # all tests
uv run pytest tests/unit/test_compiler.py::TestClass::test_method  # single test
uv run pytest -k "test_revenue"   # by name pattern

uv run ruff check src/            # lint
uv run ruff format src/ tests/    # format
uv run mypy src/                  # type check

uv sync --extra docs && uv run mkdocs serve  # docs on :8080

# Docker: two separate images (API + UI)
docker build -t orionbelt-api .                  # API-only image
docker build -f Dockerfile.ui -t orionbelt-ui .  # UI image (Gradio)

./scripts/deploy-gcloud.sh                        # Cloud Run deploy (both services)
./tests/docker/test_docker.sh                     # 15 local Docker tests
./tests/cloudrun/test_cloudrun.sh <CLOUD_RUN_URL> # 30 live API tests
```

## Code Review

Code changes are reviewed with **OpenAI Codex**. Write clean, well-structured code that passes automated review. Avoid unnecessary complexity, dead code, or patterns that would trigger review warnings. Ensure all changes pass `ruff check`, `ruff format`, and `mypy` before submitting.

## Architecture — Compilation Pipeline

`QueryObject + SemanticModel` flow through these stages, orchestrated by `CompilationPipeline` in `compiler/pipeline.py`:

1. **Resolution** (`compiler/resolution.py`) — selects base object (fact table), resolves refs, determines join paths, classifies filters; sets `requires_cfl=True` only when measures span truly independent facts (directed reachability check via JoinGraph).
2. **Fanout detection** (`compiler/fanout.py`) — raises `FanoutError` if reversed many-to-one joins would multiply rows.
3. **Planner** — `compiler/star.py` (single-fact star schema, LEFT JOINs) or `compiler/cfl.py` (multi-fact CFL, UNION ALL + NULL padding; common root per leg via `JoinGraph.find_common_root()`).
4. **Total wrap** (optional, `compiler/total_wrap.py`) — `AGG(x) OVER ()` window CTEs for measures with `total=True`.
5. → **SQL AST** (frozen dataclasses in `ast/nodes.py`).
6. **Codegen** (`compiler/codegen.py` + `dialect/*.py`) — AST → SQL string.
7. **Validate** (`compiler/validator.py`) — sqlglot post-gen check (non-blocking).

## Key Subsystems

### Dialect Registry (`dialect/`)
Dialects self-register via `@DialectRegistry.register` decorator. `dialect/__init__.py` imports all 8 modules to trigger registration. `DialectRegistry.get(name)` returns a fresh instance. Each dialect implements `quote_identifier()`, `render_time_grain()`, `render_cast()`, `current_date_sql()`, `date_add_sql()`, and `compile_expr()` (uses `match` on AST nodes).

### SQL AST (`ast/nodes.py`)
All nodes are frozen dataclasses. Key types: `Select`, `From`, `Join`, `CTE`, `UnionAll`, `ColumnRef`, `AliasedExpr`, `FunctionCall`, `BinaryOp`, `WindowFunction`, `CaseExpr`, `Cast`, `Literal`, `RawSQL`. The union type `Expr` covers all expression nodes.

### Session Management (`service/`)
`SessionManager` holds TTL-scoped sessions, each with its own `ModelStore`. Thread-safe via `threading.Lock`. Background daemon thread purges expired sessions. Default session (`__default__`) is auto-created for MCP stdio mode. REST API uses `api/deps.py` singleton pattern with FastAPI `lifespan` context manager. **Important:** `httpx.ASGITransport` does NOT trigger lifespan — tests must manually call `init_session_manager()`.

### Parser (`parser/`)
Two distinct validators exist — don't confuse them:
- `parser/validator.py` — **SemanticValidator**: validates the OBML model (cycles, duplicate names, invalid refs)
- `compiler/validator.py` — **SQL validator**: post-generation sqlglot syntax check (non-blocking warnings)

`TrackedLoader` uses ruamel.yaml for line-faithful source positions. `ReferenceResolver` converts raw dict → `SemanticModel` + `ValidationResult`.

## Pydantic v2 Alias Convention

All models use `Field(alias="camelCase")` with `populate_by_name=True`. YAML/JSON uses camelCase aliases; Python code uses snake_case field names. Mypy only sees the Python names.

Key aliases: `data_objects` → `"dataObjects"`, `join_to` → `"joinTo"`, `columns_from` → `"columnsFrom"`, `columns_to` → `"columnsTo"`, `path_name` → `"pathName"`, `use_path_names` → `"usePathNames"`, `abstract_type` → `"abstractType"`, `result_type` → `"resultType"`, `join_type` → `"joinType"`, `time_grain` → `"timeGrain"`. `DataColumnRef.view` and `Dimension.view` both alias to `"dataObject"`.

When constructing models in Python, always use the Python field names (e.g., `data_type=`, `view=`), not the aliases.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ralforion/orionbelt-semantic-layer](https://github.com/ralforion/orionbelt-semantic-layer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
