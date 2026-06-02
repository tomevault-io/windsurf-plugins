---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

VodHub is a pnpm monorepo with two applications:
- **Backend** (`apps/backend`): Hono‑based video aggregation API with a CMS proxy system and TMDB metadata integration, providing a unified REST API (categories, search, details, playback). Node.js ≥ 24, ESM.
- **Frontend** (`apps/frontend`): Vite + React 19 + React Router video player application with Ant Design 6, Zustand state management, multi‑theme support, integrated CMS management, and TMDB metadata integration.

A shared package (`packages/shared`) provides core TypeScript types used by both apps, including video source definitions.

## Common Development Commands

### Monorepo Commands (from root)
```bash
pnpm install              # Install all dependencies
pnpm dev                  # Start both backend and frontend dev servers
pnpm dev:backend          # Backend only (tsx watch)
pnpm dev:frontend         # Frontend only (vite dev)
pnpm lint                 # Lint all apps
pnpm lint:fix            # Lint with auto‑fix
pnpm typecheck           # Type check all apps
pnpm format              # Prettier write
pnpm format:check        # Prettier format check
pnpm commit              # Interactive conventional commit
pnpm build               # Build all apps (backend + frontend)
```

### Backend‑Specific Commands
```bash
pnpm --filter @vodhub/backend start     # Start backend without watch
pnpm --filter @vodhub/backend build     # Build backend (tsup)
pnpm --filter @vodhub/backend lint      # Lint backend only
pnpm --filter @vodhub/backend lint:fix  # Lint backend with auto‑fix
pnpm --filter @vodhub/backend typecheck # Type check backend only
```

### Frontend‑Specific Commands
```bash
pnpm --filter @vodhub/frontend build    # Production build
pnpm --filter @vodhub/frontend preview  # Preview production build
pnpm --filter @vodhub/frontend lint     # Lint frontend only
pnpm --filter @vodhub/frontend lint:fix # Lint frontend with auto‑fix
pnpm --filter @vodhub/frontend typecheck # Type check frontend only
```

## Backend Architecture

### Route System
- **URL Pattern**: `GET /api/vodhub/<module>/<action>`
- **Module Routes**: Explicitly registered in `apps/backend/src/app.tsx` (no auto-discovery)
- **Three route modules**:
  1. `/api/vodhub/config` – returns TMDB configuration (enabled status, API token presence)
  2. `/api/vodhub/cms` – CMS proxy system for custom video sources
  3. `/api/vodhub/tmdb` – TMDB metadata API (home, search, detail)

### Module Routes

#### Config (`/api/vodhub/config`)
- `GET /` – Returns `{ tmdb: { enabled: boolean, hasToken: boolean } }`

#### CMS Proxy (`/api/vodhub/cms`)
- `GET /proxy` – Proxies requests to user‑defined CMS URLs using `x-proxy-target` and `x-proxy-action` headers
- Supported actions: `home`, `homeVod`, `category`, `detail`, `play`, `search`
- All actions are GET requests; parameters are passed as query params
- Custom CMS URL is provided via the `x-proxy-target` header
- The `x-proxy-action` header determines which CMS handler to invoke

#### TMDB (`/api/vodhub/tmdb`)
- `GET /home` – Homepage categories (trending, popular movies/TV, now playing, upcoming, top rated, genres)
- `GET /search` – Multi-type search (`query`, `page` params)
- `GET /detail` – Movie/TV detail (`id`, `mediaType` params)

### Middleware Order
1. `cors()` – CORS handling (on cmsApp and tmdbApp)
2. `trimTrailingSlash()` – trailing slash normalization
3. `compress()` – response compression
4. `jsonReturn` – JSON response serialization
5. `cache` – Redis/memory caching with deduplication

Note: `logger()` is applied globally to the main Hono app. The config route only inherits the global logger.

### Caching System
- Two‑level cache: memory LRU + optional Redis via Keyv
- Deduplication: prevents concurrent identical requests
- Cache key format: `vod‑hub:redis‑cache:${path}${bodyHash}`
- Error responses: set `Cache‑Control: 'no‑cache'` to prevent caching failures
- GET requests with `Cache-Control: no-cache` header are not cached

### Error Handling Pattern
All CMS and TMDB handlers must follow this structure:
```typescript
try {
    logger.info(`${ACTION_MESSAGE.INFO} ‑ ${namespace.name}`);
    const res = await someRequest();
    if (res.code === 1) {  // CMS convention
        return { code: SUCCESS_CODE, message: ACTION_MESSAGE.SUCCESS, data: […] };
    }
    logger.error(`${ACTION_MESSAGE.ERROR} ‑ ${namespace.name} ‑ ${JSON.stringify(res)}`);
    return { code: ERROR_CODE, message: ACTION_MESSAGE.ERROR, data: [] };
} catch (error) {
    ctx.res.headers.set('Cache‑Control', 'no‑cache');
    logger.error(`${ACTION_MESSAGE.ERROR} ‑ ${namespace.name} ‑ ${error}`);
    return { code: SYSTEM_ERROR_CODE, message: ACTION_MESSAGE.ERROR, data: [] };
}
```

**Status Codes**:
- `SUCCESS_CODE = 0` – successful response
- `ERROR_CODE = -1` – business logic error (upstream failure)
- `SYSTEM_ERROR_CODE = -2` – exception/catch error

### Directory Structure
```
apps/backend/src/
├── api/
│   └── config/           # TMDB config API
├── modules/
│   ├── cms/proxy.ts      # CMS proxy router (single entry point for all CMS requests)
│   └── tmdb/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [consistent-k/VodHub](https://github.com/consistent-k/VodHub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
