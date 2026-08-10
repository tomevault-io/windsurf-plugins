---
trigger: always_on
description: GeoLens mixes Python and TypeScript. Backend source is in `backend/app/`: `modules/` holds domain areas, `platform/` shared services, `processing/` ingest/export/tile work, and `standards/` OGC/STAC/DCAT integrations. Migrations are in `backend/alembic/`; tests are in `backend/tests/`.
---

# Repository Guidelines

## Project Structure & Module Organization

GeoLens mixes Python and TypeScript. Backend source is in `backend/app/`: `modules/` holds domain areas, `platform/` shared services, `processing/` ingest/export/tile work, and `standards/` OGC/STAC/DCAT integrations. Migrations are in `backend/alembic/`; tests are in `backend/tests/`.

The React/Vite frontend is in `frontend/src/`: `components/`, `pages/`, `hooks/`, `stores/`, `api/`, `assets/`, `i18n/`, and colocated `__tests__/`. Playwright specs are in `e2e/`. The CLI is in `cli/geolens_cli/`; the read-only MCP server is in `mcp/geolens_mcp/`; generated SDKs are in `sdks/`; operations files are in `scripts/`, `db/`, and `.github/`.

## Build, Test, and Development Commands

- `make dev` / `make down`: start or stop the Docker Compose development stack.
- `make migrate`: run Alembic migrations in the API container. `make alembic-check` fails if the ORM models have drifted from the migration scripts (run it for schema-adjacent changes).
- `make test` / `make test-cov`: run backend pytest and coverage.
- `make ai-evals`: live-provider NL→SQL regression evals in `backend/tests/evals/` (skipped in normal test runs; costs provider tokens, needs `ANTHROPIC_API_KEY` and the dev DB).
- `npm run e2e` or `npm run e2e:smoke`: run Playwright suites.
- `cd frontend && npm ci && npm run dev`: install frontend dependencies and start Vite.
- `cd frontend && npm run build && npm run lint && npm run typecheck && npm run test:coverage`: run frontend gates (`npx tsc --noEmit` is a no-op here; `npm run typecheck` is the real type gate).
- `make openapi-check`, `make sdks-check`, `make cli-test`: validate API snapshots and SDK/CLI drift.
- `make bump VERSION=X.Y.Z`: rewrite every version site atomically. Never edit a version string by hand; `make version-check` is the CI gate.

### Running a single test

- Backend, in-container: mirror `make test`'s env (the container's default uv cache is read-only), e.g. `docker compose exec api env UV_CACHE_DIR=/app/staging/uv-cache UV_PROJECT_ENVIRONMENT=/app/staging/geolens-api-test-venv uv run pytest -o cache_dir=/app/staging/.pytest_cache tests/test_foo.py::test_bar -v`.
- Backend, on the host (needs Postgres at localhost:5434): `cd backend && set -a && source ../.env.test && set +a && uv run pytest tests/test_foo.py -v`.
- Frontend: `cd frontend && npx vitest run src/path/foo.test.ts`.
- E2E: `npx playwright test e2e/foo.spec.ts --project=chromium` (stack must be running).

A focused selection is blind to the module-size gates. `backend/tests/test_layering.py` caps the size of the largest backend modules, and CI runs it on every PR that triggers `backend-test`, so a change that adds lines to a ratcheted file passes locally and fails there. If you touched anything under `backend/app/`, finish with `cd backend && set -a && source ../.env.test && set +a && uv run pytest tests/test_layering.py -q`; it needs no database, but it does boot `app.core.config`, so a bare run dies on missing env vars with a non-zero exit before collecting anything — which reads exactly like a gate failure and is not one. In a fresh clone `.env.test` does not exist yet (it is gitignored); create it once with `cp .env.test.example .env.test` from the repo root. Growth is allowed — raise the file's cap in `_MODULE_LOC_CAPS` in the same commit, with a comment saying what the lines bought.

### Working from a git worktree

The dev stack bind-mounts the MAIN checkout (`./frontend` → `/app`, and `backend/app` → `/app/app` with `--reload`), so `localhost:8080` always serves `main` no matter which branch your worktree is on. Running `npx playwright test` from a worktree therefore validates code you did not write. Treat the result as meaningless: it has produced a false FAILURE, and the symmetric case is worse, because a worktree change that breaks e2e passes when the stack never had it. To exercise worktree app code, run Vite on the host at `:5174` with `API_PROXY_TARGET=http://localhost:8001` and point `E2E_BASE_URL` at it.

One exception: a spec-only change (editing `e2e/*.spec.ts` with no app-code change) is validly testable from a worktree, because Playwright reads the specs from your worktree while the app code stays byte-identical to `main`.

The host backend recipe above is also unrunnable verbatim from a worktree — the sandbox refuses `source` on a path outside the worktree and denies reading `.env*`. Copy `.env.test` into the worktree instead (it is gitignored; delete it when you are done) and run pytest from a wrapper script.

## Architecture

Services (`docker-compose.yml`): Nginx (prod proxy; Vite proxy in dev) fronts the FastAPI `api` (catalog, search, OGC/STAC, vector tiles) and Titiler (COG raster tiles). A `worker` runs GDAL/ogr2ogr ingestion, dispatched via the Procrastinate job queue that lives *inside* PostgreSQL (no separate broker). PostgreSQL 18 (PostGIS + pgvector + pg_trgm) is the single source of truth; object storage is MinIO/S3; Valkey is the tile/query cache.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [geolens-io/geolens](https://github.com/geolens-io/geolens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
