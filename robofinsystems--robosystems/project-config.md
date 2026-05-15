---
trigger: always_on
description: **All Python commands MUST use `uv run`:**
---

# CLAUDE.md - RoboSystems Development Guide

## Critical Rules

**All Python commands MUST use `uv run`:**

```bash
uv run python script.py    # NOT: python script.py
uv run pytest              # NOT: pytest
uv run ruff check          # NOT: ruff check
```

**Always use Docker profile `robosystems`** - never individual service profiles:

```bash
just start                 # Uses robosystems profile by default
just start robosystems     # Explicit form (same result)
                           # NOT: just start api
```

**Never use `os.getenv()` directly** - use centralized config:

```python
from robosystems.config import env
database_url = env.DATABASE_URL  # NOT: os.getenv("DATABASE_URL")
```

**Never create migrations manually** - always autogenerate:

```bash
just migrate-create "description"  # NOT: manual alembic revision
```

## API Endpoints (local testing)

Core platform APIs are mounted under `/v1` (graphs, billing, auth, etc.).
Extensions (roboledger, roboinvestor) live under `/extensions` and are
**graph-scoped at the URL level**, with three sub-surfaces:

- **Typed reads** → `POST /extensions/{graph_id}/graphql` (Strawberry + GraphiQL in dev)
- **Command writes** → `POST /extensions/{roboledger|roboinvestor}/{graph_id}/operations/{operation_name}`
- **Analytical view operations** → `POST /extensions/{domain}/{graph_id}/operations/{view_name}` — graph-backed operations that query LadybugDB rather than the extensions OLTP database. Read-only, same envelope contract as command writes. `build-fact-grid` is the first one (pivot tables over the XBRL hypercube); gated independently of the OLTP domain flags so deployments without the corresponding tenants can still mount them

All three surfaces take `graph_id` as a URL path parameter — auth + per-graph
access are validated by FastAPI dependencies before the handler runs.
GraphQL queries do NOT take a `graphId` argument; the URL is the scope.

Per-domain feature flags: `ROBOLEDGER_ENABLED` and `ROBOINVESTOR_ENABLED`
gate the corresponding GraphQL resolvers and operation routers. The
schema is built dynamically per flag combo, so a ledger-only deployment
exposes only ledger fields (no `INVESTOR_NOT_INITIALIZED` runtime errors).

**Graph lifecycle writes** follow the same CQRS pattern at
`POST /v1/graphs/{graph_id}/operations/{op_name}`:

| Operation | Path |
| --------- | ---- |
| Create subgraph | `POST /v1/graphs/{g}/operations/create-subgraph` |
| Delete subgraph | `POST /v1/graphs/{g}/operations/delete-subgraph` |
| Create backup | `POST /v1/graphs/{g}/operations/create-backup` |
| Restore backup | `POST /v1/graphs/{g}/operations/restore-backup` |
|  Change tier | `POST /v1/graphs/{g}/operations/change-tier` |
| Materialize | `POST /v1/graphs/{g}/operations/materialize` |

All graph operation responses are `OperationEnvelope` and support `Idempotency-Key`. Reads (list subgraphs, list backups, health, etc.) remain REST GETs at their existing paths.

Common mistakes:

| ❌ Wrong                              | ✅ Correct                                                                | Purpose              |
| ------------------------------------- | ------------------------------------------------------------------------- | -------------------- |
| `GET /health`, `GET /v1/health`       | `GET /v1/status`                                                           | API health check     |
| `GET /v1/ledger/{g}/entity`           | GraphQL `POST /extensions/{g}/graphql` body `{ entity { … } }`             | Ledger read          |
| `PUT /v1/ledger/{g}/entity`           | `POST /extensions/roboledger/{g}/operations/update-entity`                 | Ledger write         |
| `POST /v1/graphs/{g}/views`           | `POST /extensions/roboledger/{g}/operations/build-fact-grid`               | Fact grid query      |
| `{ entity(graphId: "kg_x") }`         | `{ entity { … } }` (graph_id comes from URL)                                | GraphQL query        |
| `GET /graphs/...`                     | `GET /v1/graphs/{graph_id}/...`                                            | Graph endpoints      |
| `POST /v1/graphs/{g}/subgraphs`       | `POST /v1/graphs/{g}/operations/create-subgraph`                           | Create subgraph      |
| `DELETE /v1/graphs/{g}/subgraphs/{n}` | `POST /v1/graphs/{g}/operations/delete-subgraph`                           | Delete subgraph      |
| `POST /v1/graphs/{g}/backups`         | `POST /v1/graphs/{g}/operations/create-backup`                             | Create backup        |
| `POST /v1/graphs/{g}/materialize`     | `POST /v1/graphs/{g}/operations/materialize`                               | Materialize graph    |

- **Root `/`** serves the Swagger UI (HTML); don't use it for health checks.
- **`/openapi.json`** is the live OpenAPI spec — useful when SDK generation drifts from the server.
- **All authenticated endpoints** take `X-API-Key` for local testing (not `Authorization: Bearer`). Read the key from `.local/config.json` after running `just demo-user`.
- **Frontend-facing auth** (JWT/Bearer) is a frontend concern; backend testing with `curl` should use the API key.

Example:

```bash
# Health check
curl http://localhost:8000/v1/status

# GraphQL read (fiscal calendar) — graph_id is in the URL, not the query

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RoboFinSystems/robosystems](https://github.com/RoboFinSystems/robosystems) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
