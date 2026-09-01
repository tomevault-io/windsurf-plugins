---
trigger: always_on
description: Automated GitHub repository discovery tool. Fetches and categorizes open-source repos by **platform** (android, windows, macos, linux) and **category** (trending, new-releases, most-popular), then commits the results as cached JSON. Runs daily via GitHub Actions.
---

# CLAUDE.md

## Project Overview

Automated GitHub repository discovery tool. Fetches and categorizes open-source repos by **platform** (android, windows, macos, linux) and **category** (trending, new-releases, most-popular), then commits the results as cached JSON. Runs daily via GitHub Actions.

## Architecture

```
scripts/
  fetch_all_categories.py   — Main fetcher (async Python, aiohttp)
  validate_releases.py      — Release date validator (sync, requests)
  requirements.txt          — Python deps: requests, aiohttp
cached-data/
  trending/{platform}.json
  new-releases/{platform}.json
  most-popular/{platform}.json
.github/workflows/
  fetch_all_categories_workflow.yml — Daily cron (2am UTC) + manual dispatch
```

## How It Works

1. `fetch_all_categories.py` reads 3 per-category tokens (`GH_TOKEN_TRENDING`, `GH_TOKEN_NEW_RELEASES`, `GH_TOKEN_MOST_POPULAR`), falling back to `GITHUB_TOKEN`
2. Each category gets its own `GitHubClient` with a dedicated token
3. If tokens are shared (same underlying user), the budget is split evenly across categories
4. For each category × platform (12 combos), checks if cached JSON is fresh (<23h)
5. If stale, queries GitHub Search API with platform-specific topics/languages/keywords
6. Filters repos that have **real releases with platform installers** — only dedicated installer formats count (no generic archives like `.zip` or `.tar.gz`)
7. Repos with NSFW/inappropriate topics or descriptions are excluded via `BLOCKED_TOPICS`
8. Verifies ALL candidates — no artificial caps. Stops gracefully when per-platform budget is exhausted or rate limit drops below `RATE_LIMIT_FLOOR` (50)
9. Never saves 0-repo results; never overwrites good cached data with poor results
10. Saves results to `cached-data/{category}/{platform}.json`
11. GitHub Actions commits and pushes changes

### Token Strategy
- 3 GitHub Classic PATs (scope: `public_repo`), each from a **separate GitHub account**
- GitHub rate limits are per-user (not per-token), so 3 accounts = 3 independent 5,000 req/hr pools = 15,000 total
- Stored as GitHub Actions repository secrets: `GH_TOKEN_TRENDING`, `GH_TOKEN_NEW_RELEASES`, `GH_TOKEN_MOST_POPULAR`
- Backward compatible: falls back to single `GITHUB_TOKEN` if per-category tokens aren't set
- If shared tokens detected, budget is automatically split evenly across categories

### Rate Limit Management

**Two independent rate limits at play:**

| Limit | Pool | Scope |
|---|---|---|
| Core API | 5,000/hr per user | Release checks, rate_limit endpoint |
| Search API | 30/min per user | `search/repositories` queries |

**Core API budget system:**
- `main()` detects shared tokens and caps each category to its fair share
- `process_category()` divides the category's budget evenly across 4 platforms
- Budget recalculates after each platform — unused budget carries forward
- `verify_installers()` stops when per-platform budget is exhausted (not just global floor)

**Search API throttling:**
- Sliding window rate limiter (`_acquire_search_slot()`) tracks timestamps of all search API calls
- Automatically pauses when approaching 28 calls per 60-second window (GitHub allows 30, 2 left as buffer)
- Pacing is per-call — no blunt inter-platform pauses needed; search calls are spaced automatically
- `_update_rate_info()` ignores search API headers to prevent core rate tracking pollution

**Safety caps:**
- `_wait_for_rate_limit()` never sleeps more than 60s (prevents workflow timeout)
- Minimum budget of 100 requests per platform regardless of remaining
- Workflow timeout: 45 minutes

### Categories
- **trending**: High star velocity + recent activity. Sorted by trending score (platform score + velocity × 10)
- **new-releases**: Repos with stable releases in last 14 days. Sorted by release date
- **most-popular**: Repos with 5,000+ stars. Sorted by star count

### Platform Detection

Each platform has: topics, installer file extensions, scoring keywords (high/medium/low), primary/secondary languages, and frameworks. See `PLATFORMS` dict.

**Installer detection** — only client-installable file extensions count (generic archives like `.zip`/`.tar.gz` are ignored). These sets MUST stay byte-for-byte equivalent to the client's `core.domain.utils.AssetPlatform` + `isAssetInstallable`, else `has_installers_*` advertises what the client refuses to install:
- Android: `.apk` **passing `is_android_apk`** — `.apk` is overloaded (nfpm/goreleaser emit Alpine Linux packages with it); a name-token discriminator (`linux`/`amd64`/`386` ⇒ NOT Android) is required. `.aab` is a Play Store bundle, not user-installable — excluded.
- Windows: `.msi`, `.exe` (`.msix` excluded — client can't install it)
- macOS: `.dmg`, `.pkg`
- Linux: `.appimage`, `.deb`, `.rpm`, `.pkg.tar.zst` (Alpine `.apk` is NOT in the client's Linux set, so an Alpine-`.apk`-only repo is installable on nothing)

### Content Filtering


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [komi-store/komi-store-backend-data](https://github.com/komi-store/komi-store-backend-data) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
