---
trigger: always_on
description: This document provides context for AI assistants working on the Memphis Concert Calendar project.
---

# Claude Code Instructions

This document provides context for AI assistants working on the Memphis Concert Calendar project.

## Project Overview

A daily-updating live music calendar for Memphis, Tennessee, built for WYXR 91.7 FM DJs.

**Architecture:**
- **Frontend:** Vercel at `concert-calendar.wyxr.org` (static HTML + interactive JS)
- **Backend API:** Flask on Render at `concert-calendar-api.onrender.com`
- **Database:** PostgreSQL on Render (single source of truth)
- **Build:** GitHub Actions 2x daily (midnight + noon Central)

## Key Technologies

- **Backend:** Python 3.12, Flask, PostgreSQL (psycopg2)
- **Frontend:** Vanilla JavaScript (no framework)
- **Deployment:** Render (backend), Vercel (frontend), GitHub Actions (CI/CD)
- **Data Sources:** Ticketmaster API, venue scrapers, Claude Vision API (for image artifacts)

## Local Development

**Before making changes, always test locally:**

```bash
# Terminal 1: Backend
./run_local_backend.sh

# Terminal 2: Frontend
./run_local_frontend.sh

# Terminal 3: Test changes
./test_before_push.sh
```

See [LOCAL_DEVELOPMENT.md](LOCAL_DEVELOPMENT.md) for complete setup.

## Critical Patterns & Lessons Learned

### Data Flow
- **PostgreSQL is the single source of truth** - never write directly to events.json
- `data/events.json` is a read-only snapshot exported by builds
- Admin edits go directly to PostgreSQL via Render API
- Scrapers merge into PostgreSQL, then export snapshot

### Database
- Use **external** PostgreSQL URL for local dev (internal URL only works on Render)
- `gunicorn preload_app=True` is critical - prevents silent worker import failures
- **psycopg2 transaction poisoning:** Use separate `with get_cursor()` blocks per operation
- Root `/` route needed for Render health checks

### Deduplication
- Events are deduplicated during scraper runs using normalized keys (title|venue|date)
- `_save_events_to_db()` tracks inserted events within same batch to prevent duplicates
- **Never overwrite admin/manual source entries** - automated scrapers always defer to manual edits

### Security
- API keys stored in: `.zshrc` (local), GitHub Secrets (CI/CD), Render env vars (production)
- `.claude/settings.local.json` contains API keys in permission strings - protected by .gitignore
- Never commit secrets - use `test_before_push.sh` to check

### File Operations
- **Never have two writers to the same file** - race conditions cause data loss
- Render storage is ephemeral - commit artifacts to GitHub, not /tmp
- GitHub artifacts are auto-cleaned after 24 hours by daily build workflow

### API Patterns
- Admin uses JWT Bearer tokens (cross-origin from Vercel to Render)
- CORS configured for `ALLOWED_ORIGINS` environment variable
- Render health checks hit `/` not `/health` - both endpoints exist

## Important Files

### Core Application
- `src/main.py` - Orchestrator (fetch → merge → prune → HTML → RSS)
- `src/config.py` - Venues, neighborhoods, keywords, date ranges
- `src/normalize.py` - Deduplication logic
- `src/generate_rss.py` - RSS 2.0 feed generator (60-day window)
- `backend/app.py` - Flask REST API
- `backend/db.py` - PostgreSQL queries

### Scrapers
- `src/sources/ticketmaster.py` - Ticketmaster Discovery API
- `src/sources/venue_scrapers.py` - Custom scrapers (Hi Tone, Minglewood, etc.)
- `src/sources/artifacts.py` - Claude Vision for image processing

### Admin UI
- `docs/admin/` - Admin interface (Events, Import, Scrapers, Venues, Sponsors tabs)
- `docs/admin/admin-common.js` - Shared admin utilities (auth, API calls)
- All admin pages use `window.__API_BASE` to point to Render backend

### Deployment
- `.github/workflows/daily.yml` - CI/CD (2x daily + manual trigger)
- `backend/gunicorn_conf.py` - Gunicorn config
- `vercel.json` - Vercel config
- `scripts/schema.sql` - PostgreSQL schema

### Feeds
- `docs/feed.xml` - RSS 2.0 feed (next 60 days, auto-generated each build)
- Feed URL: `concert-calendar.wyxr.org/feed.xml`

## Venues (15 total)

**Custom scrapers (7):** Hi Tone, Minglewood Hall, Hernando's Hideaway, Growlers (SeeTickets), Graceland (Wix), Lafayette's Music Room (Elfsight), Nashoba (Elfsight)

**Generic scraper (4):** Crosstown Arts, Overton Park Shell, FedExForum, Germantown PAC

**Manual only (4):** B.B. King's, Orpheum, Bar DKDC, B-Side Memphis

Venue scrapers use 6-month range (`SCRAPER_END_DATE`) for interactive calendar.

## Slack Image Upload Pipeline

DJs can upload venue schedule images directly to **#wyxr-concert-calendar** in Slack to add events without touching the admin UI.

### How it works
1. User uploads an image to #wyxr-concert-calendar with the caption **"add to calendar"**
2. Slack fires a `file_shared` event to `POST /api/slack/events`
3. Backend downloads the image, checks the caption via `conversations.history`
4. Claude Vision (`claude-sonnet-4-6`) extracts events from the image
5. New events are deduplicated and inserted into PostgreSQL
6. GitHub Actions rebuild is triggered
7. Bot replies in the channel with a list of added events

### Environment variables (Render)
| Variable | Source |
|---|---|
| `SLACK_BOT_TOKEN` | api.slack.com → OAuth & Permissions → Bot User OAuth Token |
| `SLACK_SIGNING_SECRET` | api.slack.com → Basic Information → Signing Secret |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wyxr-memphis) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
