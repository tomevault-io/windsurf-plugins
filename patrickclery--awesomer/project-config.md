---
trigger: always_on
description: This file provides guidance to Claude Code when working with this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project Overview

Awesomer is a multi-vertical platform for discovering trending open-source tools from curated GitHub Awesome Lists. It consists of a **NestJS API** (`api/`) and a **Next.js frontend** (`web/`), backed by PostgreSQL via Prisma ORM.

## Repository Structure

This repo is `patrickclery/awesomer` (**PUBLIC**). It contains all source code.

```
api/              → NestJS 11 backend (TypeScript, ESM, Prisma 7)
web/              → Next.js 16 frontend (React 19, Tailwind CSS 4)
```

This repo deploys the static site to GitHub Pages via the `gh-pages` branch.

Sensitive directories (`.planning/`, `.claude/`, `docs/plans/`) are excluded via `.gitignore` and never committed.

## Common Commands

### API (`api/`)
```bash
cd api

# Install dependencies
npm install

# Generate Prisma client (after schema changes)
npx prisma generate

# Create/apply migrations
npx prisma migrate dev --name description_here

# Build
npm run build                     # nest build → dist/src/

# Start (development)
npm run start:dev                 # nest start --watch

# Start (production)
npm run start:prod                # node dist/src/main.js

# Note: DATABASE_URL must be set (see .env.example)
```

### Frontend (`web/`)
```bash
cd web

npm install
npm run dev                       # Next.js dev server on port 3000 (for development)
BASE_PATH=/awesomer npm run build # Static export → out/ directory (MUST include BASE_PATH)
python3 -m http.server 3000 --directory out  # Serve static site on port 3000
```

The frontend uses `output: 'export'` for fully static generation. All pages are pre-rendered at build time. The API must be running on port 4000 during `npm run build` since `generateStaticParams()` and server components fetch data from it. The daily sync pipeline automatically rebuilds the static site after completing.

**CRITICAL: Always set `BASE_PATH=/awesomer` when building manually.** Without it, all asset paths (`/_next/...`) are missing the `/awesomer` prefix and the site renders with no CSS/JS. The sync service sets this automatically via `rebuildStaticSite()`, but manual CLI builds do not.

### Database
```bash
cd api

# Run migrations
npx prisma migrate dev

# Reset database
npx prisma migrate reset

# Open Prisma Studio (GUI)
npx prisma studio
```

### Admin Sync Endpoints

All sync endpoints require `Authorization: Bearer ${ADMIN_API_KEY}`:

```bash
# Full daily pipeline (runs async)
curl -X POST http://localhost:4000/api/sync/run -H "Authorization: Bearer $KEY"

# Re-import a specific awesome list
curl -X POST http://localhost:4000/api/sync/import/223 -H "Authorization: Bearer $KEY"

# Markdown-only regeneration
curl -X POST http://localhost:4000/api/sync/markdown -H "Authorization: Bearer $KEY"
```

## Architecture

### API Modules (`api/src/`)

| Module | Purpose | Key Endpoints |
|--------|---------|---------------|
| `prisma/` | PrismaService — wraps PrismaClient with `@prisma/adapter-pg` for Prisma 7 | — |
| `awesome-lists/` | CRUD for awesome list verticals | `GET /api/awesome-lists`, `GET /api/awesome-lists/:slug` |
| `repos/` | Repo queries, global search, star history | `GET /api/awesome-lists/:slug/repos`, `GET /api/search`, `GET /api/repos/:id/star-history` |
| `categories/` | Category browsing within a vertical | `GET /api/categories`, `GET /api/categories/:slug` |
| `trending/` | Trending repos (7d/30d/90d) from star snapshot diffs | `GET /api/trending`, `GET /api/trending/:slug` |
| `featured/` | Featured developer profiles | `GET /api/featured` |
| `sync/` | Sync pipeline + static data export for SSG build | `POST /api/sync/run`, `POST /api/sync/import/:id`, `POST /api/sync/markdown` |

### Sync Pipeline (5 Steps)

The daily sync (`POST /api/sync/run`, also runs at 2 AM UTC via cron):

1. **Import**: Fetch README from GitHub → parse markdown → upsert categories/items
2. **Stats**: REST API calls for each repo (stars, description, lastCommitAt)
3. **Snapshots**: GraphQL batch queries (100 repos/request) → upsert StarSnapshot records
4. **Trending**: Compute 7d/30d/90d deltas from snapshots (±3-day tolerance window)
5. **Markdown**: Generate markdown files (written to deploy staging directory during deployment) with Top 10 tables and per-category sections

Standalone markdown generation: `POST /api/sync/markdown` (no GitHub API needed)

### Key Technical Decisions

- **ESM throughout**: `"type": "module"` in `api/package.json`, `"module": "nodenext"` in tsconfig. All relative imports use `.js` extensions. tsc outputs ESM to `dist/src/`. Required because Prisma 7 generates ESM-only TypeScript — without `"type": "module"`, tsc outputs CJS which breaks with `exports is not defined in ES module scope`.
- **Prisma 7 client engine**: Requires `@prisma/adapter-pg` — no native engine. Uses the `prisma-client` generator (not the old `prisma-client-js`). PrismaService uses a cast pattern to extend PrismaClient (see code pattern below).
- **Generated files in `src/`**: Prisma output goes to `api/src/generated/prisma/` so tsc compiles it alongside the app. This directory is gitignored. Placing output outside `src/` causes ESM/CJS mismatch.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [patrickclery/awesomer](https://github.com/patrickclery/awesomer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
