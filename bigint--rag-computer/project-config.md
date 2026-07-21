---
trigger: always_on
description: - `api/` — Python/FastAPI backend (Docling ingestion + Turbopuffer vector search)
---

# bigRAG Platform Monorepo - Claude Instructions

## Project Structure

- `api/` — Python/FastAPI backend (Docling ingestion + Turbopuffer vector search)
- `sdks/typescript/` — TypeScript SDK (`@bigrag/client`)
- `sdks/python/` — Python SDK (`bigrag`)
- `app/` — admin UI (Vite + TanStack Router + Tailwind v4 + Base UI, `@bigrag/app`)
- `website/` — Documentation site (Next.js + Fumadocs, content in `website/content/docs/`)

## Style Guide

All coding guidelines, patterns, and conventions are documented in **[STYLEGUIDE.md](./STYLEGUIDE.md)**. Follow the rules and patterns defined there.

### No comments

Don't write comments or docstrings in code under `api/bigrag/`, `sdks/typescript/src/`, `sdks/python/src/`, `app/`, or `website/`. This includes `#`, `//`, `/* */`, `/** */` JSDoc, and Python `"""docstrings"""`. The diff and well-named identifiers should speak for themselves; surprising invariants belong in commit messages or PR descriptions, not in the code. The only allowed exceptions are functional directives — shebangs, `# type: ignore`, `# noqa`, `# ruff:`, `// @ts-…`, `// biome-ignore`, `// eslint-…`, and similar tool pragmas, plus Pydantic `Field(description="...")` strings that are load-bearing for OpenAPI.

If you find yourself wanting to explain code, rename or restructure it instead.

### One thing per file

The repo follows aggressive package-style splits. If a single file grows past ~300 LoC, look for a clean seam (per-handler, per-provider, per-stage) and split into a package directory. Examples already shipped:

- `api/bigrag/db/models/` (per domain: auth, collection, connector, document, instance, observability, preference, webhook)
- `api/bigrag/app_factory/` (lifespan, exception_handlers, routers)
- `api/bigrag/mcp/` (tools, unscoped, scoped, cli)
- `api/bigrag/services/{embedding,retrieval,webhook,vector_store,storage,url_security,access_log,event_bus,queue_conversion,queue_embedding,chat,runtime_setting_specs}/` packages
- `sdks/python/src/bigrag/resources/admin/` and `sdks/typescript/src/resources/admin/` (settings, users, api_keys, access, audit, connectors, embedding_presets, mcp_servers, vector_storage)

When adding new code, prefer the smallest meaningful module instead of dropping it into the nearest catch-all.

## Tech Stack

- **Backend**: Python 3.12+, FastAPI, SQLAlchemy 2 (async) + asyncpg, Alembic, docling, openai, cohere, cryptography (Fernet for at-rest encryption of provider secrets), dramatiq (Redis broker)
- **Vector DB**: Turbopuffer
- **Metadata DB**: PostgreSQL 17
- **Ingestion**: Docling (PDF, DOCX, PPTX, XLSX, HTML, Markdown, images)
- **Embedding**: OpenAI, Cohere, Voyage, and OpenAI-compatible providers
- **Caching/queues**: Redis (auth principal cache, embedding cache, dramatiq ingestion + webhook queues)

## Package Management

- **Python backend**: `uv` (lockfile at `api/uv.lock`)
- **Python SDK**: `uv` (lockfile at `sdks/python/uv.lock`)
- **TypeScript SDK + Website + App**: `pnpm` workspaces (root `pnpm-workspace.yaml`)

**Gotcha**: `pnpm-workspace.yaml` sets `minimumReleaseAge: 10080` (7 days). Any `pnpm install` that triggers a fresh dependency resolution may fail for packages published in the last week (commonly rolldown, rollup, and @tanstack platform binaries). Either wait, or add the offending name to `minimumReleaseAgeExclude:`.

## Linting

- **Python**: `ruff` (config in `api/pyproject.toml`)
- **TypeScript/JS**: `biome` (config in `biome.jsonc`)

**Always run lint + format before committing.** Either let the pre-commit hook run them, or invoke them manually — never commit unformatted code:

```bash
# Python (api/)
cd api && uv run ruff check --fix .
cd api && uv run ruff format .

# Python SDK
uv run --project api ruff check --fix sdks/python/src
uv run --project api ruff format sdks/python/src

# TS / JS / JSON / CSS (everything else)
pnpm exec biome check --write .
```

### Pre-commit hook

The repo ships a `.pre-commit-config.yaml` that runs `ruff check`, `ruff format`, and `biome check` on staged files. Install it once per clone:

```bash
uv tool install pre-commit   # or: brew install pre-commit
pre-commit install
```

After that, every `git commit` runs the same formatters for API Python and TS/JS/CSS files that CI enforces. Run package build/typecheck commands manually when touching SDKs. **NEVER skip hooks with `--no-verify`.** If a hook auto-fixes a file, the commit aborts — re-stage and commit again.

## Verification

The old package-level unit/integration suites, end-to-end suites, and coverage commands have been removed. Do not add package test runners, end-to-end test runners, or coverage requirements back to feature work unless the project reintroduces them deliberately.

Use lint, typecheck, build, compile, and runtime smoke checks for current verification. Keep `website/content/docs/development/testing.mdx` in sync when verification guidance changes, and do not commit generated coverage artifacts.

## Architecture Notes

- Backend uses FastAPI dependency injection via `bigrag/db/session.py::get_session` and `bigrag.middleware.auth::get_current_user` / `require_admin_session`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bigint/rag.computer](https://github.com/bigint/rag.computer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
