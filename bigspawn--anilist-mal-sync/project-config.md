---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Go CLI application that synchronizes anime/manga lists bidirectionally between AniList and MyAnimeList. Uses OAuth2 authentication and supports both YAML config and environment variables.

## Build & Development Commands

```bash
make install    # Install all dev tools (golangci-lint, gofumpt, goimports, gci, govulncheck)
make build      # Build binary: go build -o anilist-mal-sync .
make test       # Run all tests: go test ./... -v
make generate   # Generate mocks using mockgen (run before tests if interfaces change)
make fmt        # Format with gofumpt
make lint       # Run golangci-lint (new issues only)
make check      # Run all checks: generate + format + imports + vet + lint + test
make clean      # Remove artifacts and test cache
```

### Running Single Tests

```bash
go test -run TestFunctionName ./...
go test -run TestConfig ./...  # Run all tests matching "TestConfig"
```

## Architecture

### Key Abstractions

The codebase uses a **strategy pattern** for matching entries between services:

- `Source` / `Target` interfaces - abstract data from source/target services
- `StrategyChain` - chains multiple matching strategies (varies by direction and media type):
  - **Forward Anime**: Manual → ID → OfflineDB → HatoAPI → ARM → Title → APISearch
  - **Forward Manga**: Manual → ID → HatoAPI → Title → Jikan → APISearch
  - **Reverse Anime**: Manual → ID → OfflineDB → HatoAPI → ARM → Title → MALID → APISearch
  - **Reverse Manga**: Manual → ID → HatoAPI → Title → Jikan → MALID → APISearch
- `Updater` - generic orchestrator with 3-phase pipeline (resolve → deduplicate → process) that uses strategies to match and update entries
- `FavoritesSync` - separate phase for favorites synchronization that runs after the main sync, using already-resolved AniList↔MAL ID mappings

### Main Components

| File | Purpose |
|------|---------|
| `app.go` | App structure & sync orchestration |
| `cli.go` | CLI interface (urfave/cli/v3) with 6 commands: login, logout, status, sync, watch, unmapped |
| `config.go` | Config loading (env vars take precedence over YAML) |
| `oauth.go` | Token management & OAuth2 flow |
| `anilist.go` | AniList GraphQL client (via verniy library) with ToggleFavourite mutation |
| `myanimelist.go` | MAL REST API client (via go-myanimelist) |
| `anime.go` / `manga.go` | Domain models implementing Source/Target interfaces with IsFavourite field |
| `strategies.go` | Matching strategy implementations |
| `arm_api.go` | ARM API client for online ID mapping |
| `hato_api.go` / `hato_cache.go` | Hato API client for ID mapping with response caching |
| `jikan_api.go` / `jikan_cache.go` | Jikan API client for manga ID mapping with response caching and GetUserFavorites |
| `offline_database.go` | Offline database using anime-offline-database |
| `updater.go` | Generic 3-phase update orchestration (resolve, deduplicate, process) |
| `service.go` | MediaService interface and implementations |
| `favorites.go` | Favorites synchronization logic (MAL→AniList sync, AniList→MAL report only) |
| `mappings.go` | Manual AniList↔MAL mappings and ignore rules (YAML) |
| `unmapped.go` | Unmapped entries state persistence (JSON) |
| `cmd_sync.go` / `cmd_watch.go` | Sync and watch command implementations (watch supports both interval and cron schedule modes) |
| `cmd_login.go` / `cmd_logout.go` / `cmd_status.go` | Auth and status commands |
| `cmd_unmapped.go` | CLI command for managing unmapped entries |
| `report.go` | Sync report: warnings, unmapped items, duplicate conflicts, favorites mismatches |
| `statistics.go` | Sync statistics tracking and summary output with favorites results |
| `logger.go` | Leveled logger with color support, context-based logging |
| `logging.go` | HTTP round-tripper debug logging |
| `http_retry.go` | Exponential backoff retry logic |
| `docs/date-sync.md` | Documentation: date synchronization logic and behavior tables |

### Watch Modes

The `watch` command supports two mutually exclusive scheduling modes:

- **Interval mode**: fixed duration between syncs (`--interval` / `WATCH_INTERVAL` / `watch.interval`, range 1h–168h)
- **Cron mode**: sync at specific times via cron expression (`--schedule` / `WATCH_SCHEDULE` / `watch.schedule`, standard 5-field syntax)

Priority: CLI flag > env var > config YAML. Setting both modes is a validation error.

Both modes support `--once` for an immediate first sync. Cron mode uses `time.Local` timezone.

### Sync Flow

1. Load config (env vars or YAML)
2. Load manual mappings and ignore rules from `mappings.yaml`
3. Get OAuth tokens for both services
4. Fetch lists from source and target
5. **Resolve**: match entries using strategy chain (see Key Abstractions for per-direction chains)
6. **Deduplicate**: detect N:1 conflicts (multiple sources → same target), resolve by strategy priority
7. **Process**: update target service with changes
8. **Favorites sync** (if enabled with `--favorites`): synchronize favorites between services
   - MAL→AniList: add missing favorites on AniList (does not remove)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bigspawn/anilist-mal-sync](https://github.com/bigspawn/anilist-mal-sync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
