---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repo layout

- `frontend-nextjs/` is the active admin/dashboard frontend. Treat the older `frontend/` directory as legacy/reference only.
- `backend/` is a FastAPI app with SQLite persistence, Redis-backed rate limiting/cache fallbacks, and Qdrant-backed retrieval/indexing.
- `widget/` builds the embeddable chat widget SDK that talks to the backend streaming chat endpoints.
- `nginx/` contains the reverse-proxy config used in Docker deployments.
- `docker-compose.yml` is the primary local/dev/prod orchestration entrypoint.

## Common commands

### Docker compose

- Start development stack: `docker compose --profile dev up -d`
- Start production-style stack: `docker compose --profile prod up -d`
- Rebuild a service: `docker compose --profile dev up -d --build backend-dev frontend-dev`
- Follow logs: `docker compose logs -f backend-dev frontend-dev nginx`

### Frontend (`frontend-nextjs/`)

- Install deps: `npm install`
- Start dev server: `npm run dev`
- Build: `npm run build`
- Start production build locally: `npm run start`
- Lint: `npm run lint`
- Type-check: `npm run typecheck`
- Run tests: `npm run test`

### Widget (`widget/`)

- Install deps: `npm install`
- Dev bundle/example server: `npm run dev`
- Build distributables: `npm run build`
- Type-check: `npm run typecheck`
- Run tests: `npm run test`

### Backend (`backend/`)

- Install deps: `python3 -m venv venv && source venv/bin/activate && pip install -r requirements.txt`
- Run app locally: `python3 main.py`
- Run all tests: `pytest`
- Run one test file: `pytest tests/test_api.py`
- Run one test: `pytest tests/test_api.py::test_name`
- Test discovery is configured by `backend/pytest.ini` (`tests/`, `test_*.py`, `Test*`, `test_*`)
- Health check while developing locally: `curl http://localhost:8000/health`

## Architecture

### Backend request flow

- `backend/main.py` creates the FastAPI app, mounts auth plus `/api/v1` routers, configures CORS/i18n/rate limiting, and starts schedulers/Redis in non-test mode.
- CORS behavior is intentionally split between Starlette `CORSMiddleware` for normal requests and `apply_cors_headers()` from `backend/middleware/rate_limit.py` for early responses such as rate-limit/413 paths. Keep those in sync via the shared helper; do not add ad-hoc CORS header logic elsewhere.
- `Origin: null` is only allowed when `cors_allow_null_origin` is explicitly enabled in config; missing `Origin` headers should not receive wildcard CORS.
- `backend/config.py` centralizes settings. Secrets can come from env vars or on-disk key files; missing/insecure `SECRET_KEY` values are auto-generated and persisted. The default widget agent ID is also persisted to `/app/data/.agent_id`, and can be overridden with `DEFAULT_AGENT_ID`.
- `backend/database.py` sets up the async SQLAlchemy engine/sessionmaker and initializes default workspace/agent data using the configured persistent default agent ID.
- `backend/models.py` is the system-of-record schema: workspace/agent config, URL and QA knowledge sources, document chunks, chat sessions/messages, quotas, index jobs, and admin users.

### Chat, RAG, and indexing

- Main chat APIs live in `backend/api/v1/endpoints.py`. They handle admin config APIs, public chat APIs, SSE streaming, session creation, quota checks, widget origin whitelist checks, and source normalization.
- URL and Q&A ingestion lives in `backend/api/v1/url_endpoints.py`. Those routers are admin-protected at the router level; URL creation queues async fetch jobs, and refetch/crawl/import operations feed the same knowledge-source tables.
- Full index rebuilds live in `backend/api/v1/index_endpoints.py`. Those routes are also admin-protected at the router level; rebuild jobs chunk URL/QA content, persist `DocumentChunk` rows, and replace the agent’s Qdrant collection.
- Retrieval/storage logic is split across `backend/services/qdrant_store.py`, `backend/services/rag_qdrant.py`, `backend/services/scraper.py`, `backend/services/crawler.py`, and `backend/services/llm_service.py`.
- URL safety/SSRF checks are centralized in `backend/services/url_safety.py` and reused by both schema validation and scraper fetch/discovery flows. If URL-ingestion policy changes, update the shared safety helper rather than reintroducing regex-only validation in multiple places.
- `backend/services/llm_service.py` is the provider abstraction layer. Provider selection is driven by `Agent.provider_type`; many providers are implemented via OpenAI-compatible base URLs, while OpenAI Native and Google have dedicated paths.
- Task concurrency for fetch/rebuild operations is guarded by the shared task lock service used by the URL and index endpoints.

### Frontend structure

- The Next.js app uses the App Router under `frontend-nextjs/app/`, with route groups for auth pages and dashboard pages.
- Most page logic is delegated into `frontend-nextjs/src/views/`; shared UI/components live in `frontend-nextjs/src/components/`.
- `frontend-nextjs/src/context/AuthContext.tsx` stores admin auth state in `localStorage` and powers `RequireAuth`-guarded dashboard routes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [haoyiyin/basjoo](https://github.com/haoyiyin/basjoo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
