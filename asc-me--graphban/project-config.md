---
trigger: always_on
description: Graphban is an agent-native dev tool: linear tracker + pgvector agent memory +
---

# Graphban — agent guide

Graphban is an agent-native dev tool: linear tracker + pgvector agent memory +
request triage + a code-structure graph, all operable by coding agents through
57 MCP tools (`POST /api/mcp`, JSON-RPC) that share one service layer with the
REST API and web UI. Local-Docker-first; stays fully offline by default (stub
embeddings/chat — real providers are opt-in env config).

This file is a map, not a manual. Deeper truth lives in [`docs/`](docs/README.md);
read the route for your task class, not the whole corpus.

## Operating loop

Every change, regardless of task class:

```bash
# Backend (from backend/; venv via `uv venv --python 3.12 .venv && uv pip install -e ".[dev]"`)
./.venv/bin/python -m pytest -q          # SQLite, ~45s. pytest is NOT on the host PATH.

# Backend against real Postgres+pgvector (what production runs — CI runs both):
docker run -d --name gb-pg -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_DB=graphban_test -p 5544:5432 pgvector/pgvector:pg16
DATABASE_URL="postgresql+psycopg://postgres:postgres@localhost:5544/graphban_test" \
  ./.venv/bin/python -m pytest -q        # also proves the Alembic chain from empty

# Frontend (from web/):
pnpm test && pnpm typecheck              # build with `pnpm build`

# Fleet supervisor (from fleet/) — a SEPARATE distribution with its own venv (PRD-22 D-e).
# Only needed when touching fleet/ or backend dependencies, which its thin-install guard reads.
uv venv --python 3.12 .venv && uv pip install -e ".[dev]"
./.venv/bin/python -m pytest -q          # refuses to run against an uninstalled tree

# Full stack:
docker compose up --build                # web :8080, api :8000; starts EMPTY by design
```

CI (`.github/workflows/ci.yml`) runs each of these on every PR, gated by
`dorny/paths-filter` so a web-only PR does not pay for two backend suites. A change is not done
until both database engines pass — SQLite and Postgres have separate vector-search
implementations (`services/memory.py`, `services/code_graph.py`) and only the
Postgres run executes the real `<=>` SQL and migrations.

**Then run it against real data before calling it done.** Not a bonus pass — the
highest-yield check available, and the one a green suite cannot substitute for. On
2026-08-08/09 it found four defects in a row that every test missed: a completeness
pass reporting `## Decisions from grilling` as a missing feature, a close report
telling a PM that "Problem" and "Goals" were never delivered, a classifications view
returning an empty list where the truth was "ten items nobody looked at", and a
separation-of-duties check reporting a clean pass because nothing recorded who did
the work. Each looked correct in tests and was obviously wrong in the first real
output. Deploy, then read what it actually says about a real PRD.

## Invariants (violating these is the review comment you'll get)

- **One service layer.** MCP tools, REST routers, and anything new call the same
  functions in `backend/app/services/`. Never duplicate domain logic in a router
  or tool handler.
- **Schema is owned by Alembic** on Postgres (`backend/alembic/versions/`,
  currently 0001–0129). SQLite/tests use `create_all`. Never edit an applied
  migration; add a new one.
- **AI providers only via `backend/app/providers/`** (`Embedder`/`ChatModel`/
  `Extractor` protocols, selected by `EMBED_PROVIDER`/`CHAT_PROVIDER`). Offline
  stub is the default; cloud deps stay lazy imports behind the `cloud` extra.
- **Frontend data access only via `web/src/lib/api.ts` + `queries.ts`**
  (TanStack Query). Query keys include the active project id.
- **Enums live in services** (`services/items.py:STATUSES`, `requests.py`,
  `links.py`, `code_graph.py`) — reference them; don't inline copies.

## Design defaults (weaker than invariants; argue with them if you have a reason)

- **Simplest thing that fully meets the requirement.** No speculative
  abstraction, configuration, or indirection. Config is the expensive one: 49
  settings already exist, and each new one is a permanent branch in every
  deployment's behaviour. PRD-14 proposed a per-project `profile` field and it
  was cut for exactly this — the need was imagined, and removing it made the
  design better, not smaller.
- **Grow in layers; never trade a working product for unfinished complexity.**
  Decompose so the first item is the smallest thing that works end to end and
  each later one lands on something that already runs. In practice: the root
  item ships first, the acceptance walk ships last, and every PR in between
  leaves `main` deployable.
- **Reach for what's already here.** Use the dependencies the project already
  has before adding one or writing your own — and check a library's docs and
  types before concluding it can't do the thing. Reimplementing is allowed, but
  the reason goes in a comment (see `_validate_args` in `mcp_server.py`, which
  says why it hand-rolls schema checking).
- **An absence must not read as a clean result.** When you report nothing — an empty
  list, a `false`, a zero, a null — ask whether it can also mean "nobody looked". If
  it can, it needs a third answer, because the quiet reading is always the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [asc-me/graphban](https://github.com/asc-me/graphban) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
