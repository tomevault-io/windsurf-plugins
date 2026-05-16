---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Workflow

- Only commit or push when the user explicitly asks in the current message (e.g., "commit", "push", "commit and push"). Never carry forward commit/push intent from earlier messages.

## Project Overview

MovieCal is a TypeScript web scraper that collects movie screening times from Vancouver cinema websites, stores them in PostgreSQL, and displays them in a timeline web interface.

## Commands

```bash
npm install          # Install dependencies
npm run build        # Compile TypeScript to dist/
npm start            # Run compiled JavaScript from dist/
npm run scrape       # Run full scraping job (all venues + TMDB + DB save)
npm run migrate      # Run database migrations
npm run repair       # Re-clean titles + backfill missing TMDB data
npm run clear        # Clear all data from database
npm run drop         # Drop all tables from database
npm run server       # Start web server on http://localhost:3000
```

## Deployment

Deployed on Fly.io as `movieclock`.

**Auto-deploy**: Pushing to `main` triggers GitHub Actions (`.github/workflows/deploy.yml`) which runs `flyctl deploy --remote-only`. Do NOT run `fly deploy` manually — just push and GHA handles it.

**Manual commands** (for debugging/repair only):
```bash
fly status -a movieclock                      # Check app status
fly ssh console -a movieclock -C "command"   # Run command on server
fly logs -a movieclock                        # View logs
```

**Important**: `tsx` is not available on the production image (it's a dev dependency). To run scripts on prod, use the compiled JS: `fly ssh console -a movieclock -C "node dist/db/repair.js"` (NOT `npm run repair`).

**Configuration** (`fly.toml`):
- `min_machines_running = 1` - Keeps one machine always running for cron jobs
- Release command runs migrations on deploy (`node dist/db/migrate.js`)

## Architecture

### ES Modules Configuration

- Project uses `"type": "module"` in package.json
- **CRITICAL**: All imports must include `.js` extension, even when importing `.ts` files
  - Example: `import { Movie } from './models.js'` (NOT `'./models'` or `'./models.ts'`)
- TypeScript compiles `.ts` → `.js` but import statements must already reference `.js`

### Entry Points

1. **`src/scrape.ts`** - Production scraping job
   - Runs all scrapers in parallel (VIFF, Rio, Cinematheque, Park, Hollywood, Cineplex)
   - Re-cleans existing movie titles (see Title Cleaning below)
   - Enriches new movies with TMDB and Letterboxd data
   - Saves to PostgreSQL using delete-and-reinsert per scraper
   - Use `npm run scrape` to run, or `npm run scrape {name}` for a single scraper

2. **`src/server.ts`** - Hono web server
   - Routes requests to page renderers
   - Hosts admin API endpoints for TMDB/Letterboxd fix-match
   - Runs cron job every 2 hours to scrape
   - After 10pm Pacific, home page auto-shows tomorrow's screenings
   - Runs on port 3000

### Web Pages

Page rendering is in `src/pages/`:

- **`src/pages/index.ts`** - Home page (`/`)
  - Desktop: Timeline view with theatre rows and time-positioned screening blocks
  - Mobile (<800px): Listing view with theatre sections listing movies
  - Cineplex venues collapse multiple auditoriums into one group
  - Query by date: `/?date=YYYY-MM-DD`

- **`src/pages/movie.ts`** - Movie detail page (`/movie/:id`)
  - Shows poster, title, year, runtime, director, TMDB + Letterboxd links
  - Chronological list of future screenings with notes displayed
  - Hidden TMDB fix-match modal (10 clicks on poster to activate, requires `ADMIN_TOKEN`)

- **`src/pages/theatre.ts`** - Theatre detail page (`/theatre/:name`)
  - Lists all future screenings at this theatre with notes displayed

- **`src/pages/movies.ts`** - Movies page (`/movies`)
  - All movies with upcoming screenings, grouped by movie
  - Client-side sort: Date Added, Name, Popularity (TMDB)
  - Client-side theatre filtering (persisted in localStorage)

- **`src/pages/all-movies.ts`** - Internal movies page (`/internal-movies`)
  - Admin-oriented movie list with TMDB fix-match modal

- **`src/pages/layout.ts`** - Shared page layout/shell
  - Nav bar with search (searches movies with upcoming screenings)
  - Cloudflare Web Analytics

- **`src/pages/tmdb-modal.ts`** - Shared TMDB fix-match modal component

### API Endpoints

- `GET /api/movie/:id/tmdb-search` - Search TMDB (requires `ADMIN_TOKEN`)
- `POST /api/movie/:id/tmdb-update` - Fix TMDB match for a movie (requires `ADMIN_TOKEN`)
- `POST /api/movie/:id/letterboxd-update` - Fix Letterboxd URL for a movie (requires `ADMIN_TOKEN`)
- `GET /robots.txt` - Robots file with sitemap reference
- `GET /sitemap.xml` - Dynamic sitemap of movies and theatres with future screenings

### Database Layer

**Stack**: PostgreSQL + Kysely (type-safe SQL query builder) + pg driver

**Schema** (defined in `migrations/*.sql`, types in `src/db/schema.ts`):

`movie` table:
- `id`, `title`, `year`, `director`, `runtime`
- `tmdb_id`, `tmdb_url`, `poster_url`, `tmdb_popularity` — from TMDB
- `letterboxd_url` — from Letterboxd (`'MISS'` = searched but not found, `null` = not yet searched)
- `created_at`, `updated_at`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jordanhudson/moviecal](https://github.com/jordanhudson/moviecal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
