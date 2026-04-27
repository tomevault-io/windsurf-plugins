---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Full-stack monorepo (React 19 + Django 6) deployed on Mac Mini via Docker + Cloudflare Tunnel.

- **Frontend**: http://localhost:5173 (Vite, NOT port 3000). Build output: `build/` (NOT `dist/`). Import alias `@` → `src/`
- **Backend**: http://localhost:8000. Single app: `api/`. Uses **uv** (NOT pip)
- **Dev proxy**: Vite proxies `/api` → `http://127.0.0.1:8000` (no CORS issues in dev)
- **Node ≥ 24**, **Python 3.12** required. Husky pre-commit runs lint-staged automatically

> See `.claude/rules/strategy.md` for the rationale behind key architecture decisions and documented LLM failure patterns from this repo's history (incomplete refactors, hallucinated versions, confirmation bias loops, etc).

> **CLAUDE.md is the single source of truth** for operational rules. README has a brief summary pointing here. Do NOT duplicate these sections back into README.

## Commands

All `npm run` frontend commands run from `frontend/`. Root-level `npm run dev` runs both servers.

```bash
npm run dev                # Frontend + Backend (from root)
npm run dev:clean          # Kill ports first, then start both (from root)
npm run build              # sitemap → tsc → vite build → cp 404.html (from frontend/)
npm run build:no-prerender # Same but skips prerender step (from frontend/)
npm run validate           # lint + type-check + test:coverage (from frontend/)
CI=true npm test -- --run src/components/common/__tests__/Navbar.test.tsx  # Single test
npm run test:e2e           # Playwright E2E (from frontend/). Also: test:e2e:ui, test:e2e:debug
npm run type-check         # tsc --noEmit (from frontend/)
npm run analyze:bundle     # source-map-explorer (from frontend/, requires build first)

npm run knip                   # Dead code detection (unused files, exports, deps). Run from root
uv sync --extra dev                    # Install backend deps (NOT --dev). Run from backend/
uv run python manage.py test           # Django unittest (NOT pytest). Needs DATABASE_URL=""
uv run python manage.py test api.tests.CategoryAPITestCase.test_list  # Single backend test
uv run black . && uv run flake8 .      # Format + lint (line length 120)

# Make shortcuts (run from root):
# make install          # npm install + uv sync --extra dev
# make dev              # Docker dev servers (start-dev.sh)
# make dev-local        # Local dev (npm run dev — no Docker)
# make dev-clean        # Kill ports first, then local dev
# make test             # Frontend + backend concurrently (concurrently)
# make test-ci          # CI mode (no watch, with coverage report)
# make lint / lint-fix  # Check / auto-fix both frontend + backend
# make build            # docker compose build
# make down / restart   # Stop / restart Docker services
# make logs             # docker compose logs -f
# make logs-security    # Backend security.log (auth, IP blocks, rate limits)
# make logs-debug       # Backend debug.log (requests, errors)
# make ps               # Docker service status
# make migrate          # Run Django migrations in Docker
# make shell            # Django shell in Docker
# make createsuperuser  # Create admin user in Docker
# make clean            # Remove build artifacts + __pycache__
# make cleanup-visits DAYS=90  # Delete old SiteVisit records
# make setup-cron       # Install daily 3 AM SiteVisit cleanup cron
# make update-test-counts     # Run Vitest/Django and rewrite README test counts (use when count drift fails CI)
```

## Architecture

System shape: providers, data flow, routing, and module boundaries.

**i18n routing**: Korean default (no prefix: `/profile`), English `/en/profile`. Internal links must use `useLocalizedPath` hook — never raw `navigate()`/`<Link>`. Non-React data files must use getter functions (not module-level constants) so `i18n.t()` resolves at call time.

**Provider order** (in `App.tsx`): HelmetProvider → ErrorBoundary → UIProvider → AuthProvider → NotificationProvider → BlogProvider → RouterProvider.

**Auth**: JWT via httpOnly cookies (not localStorage). `auth_hint` flag in localStorage prevents 401 spam — if unset, `getUser()` is skipped on mount.

**State management**: React Context only — `UIContext` (theme/sidebar), `AuthContext` (JWT user), `BlogContext` (posts/categories), `NotificationContext` (alerts). No Redux or external state libraries.

**API layer**: `services/api.ts` (Axios) with interceptors for JWT refresh. Mock API uses `mockResponse<T>(data, status?, statusText?)` helper to avoid boilerplate. Tests stub HTTP via `vi.mock('axios')` in individual test files — there is **no** MSW server. (A `test-utils/mocks/` MSW scaffold existed for years but was never wired into `setupTests.ts`; removed in the 2026-04-11 stability sweep along with the `msw` and `terser` packages — both dead-but-installed deps that generated dependabot churn.)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/researcherhojin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
