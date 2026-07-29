---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Watchly is a Stremio catalog addon that generates personalized movie/series recommendations from a user's watch history. It is a FastAPI service that speaks the Stremio addon protocol (manifest + catalog endpoints). Recommendations come from a taste profile built off the user's history, then candidates are pulled from TMDB / Simkl, scored, capped for diversity, enriched, and returned as a Stremio catalog.

A user installs Watchly through its `/configure` web page: they paste a Stremio email/password (or auth_key), optionally connect Trakt and/or Simkl via OAuth, optionally provide their own TMDB / Gemini / Simkl / RPDB API keys, pick which catalogs they want, and get an addon manifest URL to paste into Stremio. From then on, every catalog row in their Stremio home — "Top Picks for You", "Because you loved …", "Genre & Keyword Catalogs", etc. — is served by this app. State per user is keyed on a short opaque token embedded in the manifest URL; credentials are encrypted at rest in Redis. The app must work for users who store their library in Stremio, in Trakt, or in Simkl, and for users with mixed signals (rated, watched, loved, rewatched). That source flexibility is the central architectural constraint.

## Commands

Dependencies are managed with [uv](https://github.com/astral-sh/uv); a `requirements.txt` is also kept in sync for non-uv environments. Python 3.12+.

```bash
# Install
uv sync

# Run dev server (auto-reload when APP_ENV=development)
uv run main.py --dev
# or directly
uvicorn app.core.app:app --reload

# Tests (pytest is not in requirements-dev.txt — install once into the venv)
pip install pytest pytest-asyncio
pytest tests/                                       # all tests
pytest tests/test_catalog_endpoint.py -v            # single file
pytest tests/test_catalog_endpoint.py::test_name    # single test

# Lint / format (also runs on commit via pre-commit)
pre-commit run --all-files
black .            # line length 120, py312
isort .            # black profile
flake8 .           # max-line-length 120, config in setup.cfg

# Docker
docker-compose up -d            # uses env_file .env
```

The configure UI is served at `/configure`. Required env vars: `TMDB_API_KEY`, `TOKEN_SALT`, `HOST_NAME`. Redis is required (`REDIS_URL`).

## Architecture

### Request flow

Every catalog request resolves through one path:

1. **`app/services/context.py:load_user_context`** is the entry point for every authenticated endpoint. It reads the encrypted token from Redis, decrypts credentials, parses `UserSettings`, resolves a Stremio `auth_key`, and builds the `LibraryCollection`. The library is sourced from `user_settings.watch_history_source` — `"stremio"`, `"trakt"`, or `"simkl"`. For external sources the WatchHistory is converted to a `LibraryCollection` (rating ≥ 9 → loved, 7–8.9 → liked, no-rating + rewatch → loved fallback, else watched) so downstream catalog code is source-agnostic. The `LibraryCollection.source` field drives cache invalidation when a user switches sources.
2. **`app/services/recommendation/catalog_service.py`** routes the catalog ID to one of the recommendation engines:
   - `watchly.rec` → `TopPicksService` (combines profile-driven Discover + library-seeded TMDB/Simkl recs)
   - `watchly.theme.*` → `ThemeBasedService` (genre/keyword/era driven)
   - `watchly.item.*` → `ItemBasedService` (seeded by a single library item — see "watchly.item" below)
   - `watchly.creators` → `CreatorsService` (directors/cast)
   - `watchly.all.loved`, `watchly.liked.all` → `AllBasedService`
3. The engine returns a list of items that are passed through metadata enrichment (`app/services/recommendation/metadata.py`), poster ratings overlay (`app/services/poster_ratings/`), translation, and serialization.

### Taste profile pipeline (`app/services/profile/`)

The `TasteProfile` is a numerical fingerprint of the user — top genres, keywords, directors, cast, eras, countries, runtime preference. It is built from the same source as the library: `ProfileService.build_and_cache_profile` checks the configured `watch_history_source` and feeds `WatchHistoryItem`s through the same vectorizer pipeline regardless of origin. Profiles are cached in Redis per-token-per-content-type and invalidated when the source field doesn't match. `_build_from_external_source` reuses the already-built `LibraryCollection` when its `source` matches the configured source, avoiding a duplicate Trakt/Simkl fetch.

### External API clients

All HTTP calls go through **`app/core/base_client.py:BaseClient`**, which provides retries (with jitter on 429/5xx), timeouts, structured error logging, and safe JSON parsing. `TraktService`, `SimklService`, and `TMDBService` are singletons that wrap `BaseClient`. The token-refresh + 401-revoke flow for Trakt/Simkl lives in `ProfileService.fetch_external_watch_history` and is shared between context loading and profile building.

### Caching (`app/services/user_cache.py`, `app/services/redis_service.py`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TimilsinaBimal/Watchly](https://github.com/TimilsinaBimal/Watchly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
