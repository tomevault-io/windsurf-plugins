---
trigger: always_on
description: This project uses pnpm, not npm.
---

# Repository Guidelines

## Frontend dev

This project uses pnpm, not npm.

### Managing AI-Generated Planning Documents

AI assistants often create planning and design documents during development:

- PLAN.md, IMPLEMENTATION.md, ARCHITECTURE.md
- DESIGN.md, CODEBASE_SUMMARY.md, INTEGRATION_PLAN.md
- TESTING_GUIDE.md, TECHNICAL_DESIGN.md, and similar files

**Best Practice: Use a dedicated directory for these ephemeral files**

**Recommended approach:**

- Create a `history/` directory in the project root
- Store ALL AI-generated planning/design docs in `history/`
- Keep the repository root clean and focused on permanent project files
- Only access `history/` when explicitly asked to review past planning

**Example .gitignore entry (optional):**

```
# AI planning documents (ephemeral)
history/
```

**Benefits:**

- ✅ Clean repository root
- ✅ Clear separation between ephemeral and permanent documentation
- ✅ Easy to exclude from version control if desired
- ✅ Preserves planning history for archeological research
- ✅ Reduces noise when browsing the project

## Project Structure & Module Organization

`backend/` hosts the FastAPI service; routers sit in `app/api`, config in `core`, persistence helpers in `db`, domain models in `models`, payloads in `schemas`, and business logic in `services`, with `main.py` as the uvicorn entry point. `frontend/src` stays feature-first (`api`, `components`, `features`, `pages`, `hooks`, `lib`, `types`). Dockerfiles plus the root `docker-compose.yml` wire Postgres, backend, and the nginx React build.

## Build, Test, and Development Commands

- `cd backend && python -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt` — install runtime deps.
- `cd backend && poetry install` — optional, but grabs dev tools (pytest, Ruff) defined in `pyproject.toml`.
- `cd backend && uvicorn app.main:app --reload` — run the API on http://localhost:8000.
- `cd backend && alembic upgrade head` — apply the latest database migrations (or run `python -m app.db.init_db` to migrate plus seed defaults).
- `cd backend && alembic revision --autogenerate -m "desc"` — generate a migration after SQLModel changes.
- `cd frontend && npm install && npm run dev` — launch the Vite dev server (uses `VITE_API_URL`, defaults to `http://localhost:8000/api/v1`).
- `docker-compose up --build` — start Postgres 17, backend, and the nginx SPA.
- `cd backend && pytest` / `ruff check app` and `cd frontend && npm run lint` — run tests and linters.

## Generated API Types (Orval)

Frontend TypeScript types and React Query hooks in `frontend/src/api/generated/` are auto-generated from the backend's OpenAPI spec using Orval. **Do not hand-edit these files.**

- `frontend/src/types/api.ts` re-exports all generated types and adds backward-compatible aliases (e.g., `Task = TaskListRead`).
- Generated files are committed to the repo so the frontend builds without a running backend.

**After changing backend schemas** (`backend/app/schemas/`), regenerate:

```bash
# Option 1: With a running backend
cd frontend && pnpm generate:api

# Option 2: Without a running backend
cd backend && python scripts/export_openapi.py ../frontend/openapi.json
cd frontend && pnpm orval
```

Always commit the regenerated output. CI enforces this — the `check-generated-types` job will fail if generated files don't match the current backend schemas.

**Key files:**
- `frontend/orval.config.ts` — Orval configuration
- `frontend/src/api/mutator.ts` — Axios wrapper that preserves auth/guild interceptors
- `frontend/scripts/generate-api.sh` — Generation script (supports `--from-spec <path>` for CI)
- `backend/scripts/export_openapi.py` — Exports OpenAPI JSON without a running server

## Versioning

This project uses **semantic versioning** (semver) with a single source of truth: the `VERSION` file at the project root.

### How Versioning Works

- **Single source**: The `VERSION` file contains the current version (e.g., `0.1.0`)
- **Backend**: Reads VERSION file and exposes via `/api/v1/version` endpoint and OpenAPI schema
- **Frontend**: Vite injects VERSION as `__APP_VERSION__` constant, displayed in the sidebar footer
- **Docker**: VERSION is copied into the image and set as OCI labels

### Releasing a Version

Releases are managed by `scripts/promote.sh`, which creates a PR from `dev` to `main` with the version bump and changelog stamp. Only code owners (@jordandrako, @LeeJMorel) can run this script.

```bash
# Patch release (0.29.1 → 0.29.2)
./scripts/promote.sh --patch

# Minor release (0.29.1 → 0.30.0)
./scripts/promote.sh --minor

# Major release (0.29.1 → 1.0.0)
./scripts/promote.sh --major

# Preview without making changes
./scripts/promote.sh --dry-run
```

After the release PR merges to `main`, `tag-release.yml` auto-creates the version tag, which triggers the Docker build and GitHub Release.

### Semantic Versioning Guidelines

- **MAJOR** (1.0.0): Breaking changes, incompatible API changes
- **MINOR** (0.2.0): New features, backward-compatible additions
- **PATCH** (0.1.1): Bug fixes, backward-compatible fixes

### Changelog Maintenance

**IMPORTANT**: Always update `CHANGELOG.md` when making significant changes.

**Determining where to add changes:**

1. Check git log for recent "bump version to X.Y.Z" commits

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Morelitea/initiative](https://github.com/Morelitea/initiative) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
