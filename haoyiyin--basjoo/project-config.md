---
trigger: always_on
description: This is the primary reference for AI coding agents (Pi, Claude, Cursor, Aider, etc.) working on the Basjoo repository. **Always read this file, CLAUDE.md, and relevant sections of README.md before starting any task.** Implementation plans are in `docs/plans/`; capability specs are in `docs/specs/`.
---

# AGENTS.md

## Scope
This is the primary reference for AI coding agents (Pi, Claude, Cursor, Aider, etc.) working on the Basjoo repository. **Always read this file, CLAUDE.md, and relevant sections of README.md before starting any task.** Implementation plans are in `docs/plans/`; capability specs are in `docs/specs/`.

## Project overview
Docker-oriented AI customer support platform:
- FastAPI backend with self-developed multi-tenant KB (Qdrant-backed RAG), streaming chat (SSE), knowledge ingestion, admin auth, quotas.
- Next.js 14 (App Router) admin dashboard in `frontend-nextjs/`.
- Embeddable TypeScript widget in `widget/` (localStorage sessions, SSE, human takeover).
- Supporting: Scrapling microservice, Qdrant (vector DB), Redis, PostgreSQL, nginx.
All LLM calls to external providers; embeddings via self-KB (Jina/SiliconFlow/OpenAI-compatible).

## Repository layout
- `backend/` — FastAPI app, `services/` (logic), `api/` (thin routers), `models.py`, `tests/`.
- `frontend-nextjs/` — `app/` (routes), `src/views/`, `src/components/`, `src/hooks/`, `src/services/api.ts`.
- `widget/` — `src/BasjooWidget.tsx`, esbuild bundles, example/.
- `scrapling-service/` — standalone stealth scraper (curl_cffi + readability).
- `docker-compose.yml` — dev/prod profiles; `nginx/`.
- `tests/e2e/` — Playwright specs.
- `docs/` — `plans/` (implementation plans), `specs/` (capability specs).

## Required tools and setup
- Dev stack: `docker compose --profile dev up --watch`.
- Backend local: `cd backend && python3 -m venv venv && source venv/bin/activate && pip install -r requirements.txt && python3 main.py`.
- Frontend: `cd frontend-nextjs && npm install`.
- Widget: `cd widget && npm install`.
- Environment: documented in `.env.example`; `SECRET_KEY`, `ENCRYPTION_KEY`, `DEFAULT_AGENT_ID` auto-persisted to `/app/data/` (preserve volume in prod). Never commit secrets.

## Development commands
- Frontend dev: `cd frontend-nextjs && npm run dev`.
- Frontend verify: `cd frontend-nextjs && npm run build && npm run typecheck && npm run test`.
- Widget dev: `cd widget && npm run dev`.
- Widget build (typecheck + bundles): `cd widget && npm run build`.
- Sync widget to backend: `npm run sync-widget`.
- Backend tests: `cd backend && pytest` (uses `pytest.ini`; `Test*` classes, `test_*` funcs).
- E2E (smoke, dev env): `npm run test:e2e` (auto-starts docker compose --profile dev)
- E2E (prod-like): `npm run test:e2e:prod` (requires docker compose --profile prod up -d first)
- E2E (all): `npm run test:e2e:all`
- E2E (widget): `npm run test:e2e:widget`
- Docker rebuild: `docker compose --profile dev up -d --build <service>`.
- Health: `curl http://localhost:8000/health`.

## Must-follow conventions
- **Structure**: Backend logic strictly in `backend/services/`; thin routers in `backend/api/`. Models in `backend/models.py`. Frontend views in `src/views/`, shared in `src/components/`, hooks in `src/hooks/`. Widget self-contained in `widget/src/`.
- **Style**: Python — 4 spaces, snake_case for modules/functions/tests. TypeScript/React — 2 spaces, PascalCase for components/views, `use*` hooks. Explicit TS types; no `any`.
- **Commits**: Conventional (`feat:`, `fix:`, `docs:`), scoped, imperative. PRs require summary, test commands+output, UI screenshots, migration notes.
- **Security**: Route all URLs through `backend/services/url_safety.py` (SSRF + DNS cache). Widget origin whitelists enforced. Handle CORS/rate-limit via shared middleware helpers.

## Architecture boundaries
- `backend/main.py` owns app factory, middleware (CORS, i18n, rate-limit, body-size), router mounting (`/api/admin`, `/api/v1`), scheduler/Redis startup.
- Self-KB integration (`kb_service.py`, `qdrant_service.py`, `kb_document_processor.py`): tenant-scoped document upload/parse/chunk/embed via Qdrant. Per-tenant collections. Similarity search; default similarity_threshold 0.01.
- Task concurrency guarded by shared TaskLock in URL/index endpoints.
- Widget auto-detects `apiBase` from `<script src>`; persists visitor/session in localStorage; polls for human takeover.

## When changing areas
- **New LLM provider**: Extend `backend/services/llm_service.py`, update Agent model/config, expose in Playground UI.
- **New knowledge source type**: Extend ingestion via `backend/api/v1/kb_document_endpoints.py` + `services/kb_document_processor.py` + `services/document_parser.py` (local storage + Qdrant, tenant-scoped).
  - For multi-tenant KB documents: use the new direct pipeline in `backend/api/v1/kb_document_endpoints.py` + `services/kb_document_processor.py` + `services/document_parser.py` (local storage + Qdrant, tenant-scoped).
- **UI change**: Update `src/views/` or `src/components/`; add i18n strings in `src/locales/`; verify responsiveness.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [haoyiyin/basjoo](https://github.com/haoyiyin/basjoo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
