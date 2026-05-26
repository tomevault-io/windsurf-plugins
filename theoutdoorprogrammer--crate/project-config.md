---
trigger: always_on
description: Self-hosted music manager (Lidarr alternative). Search for artists via pluggable gRPC providers (MusicBrainz, Deezer, or custom), watch their discographies, and automatically download tracks via slskd (Soulseek).
---

# Crate

Self-hosted music manager (Lidarr alternative). Search for artists via pluggable gRPC providers (MusicBrainz, Deezer, or custom), watch their discographies, and automatically download tracks via slskd (Soulseek).

## Architecture

```
cmd/crate/main.go              Entry point. Wires services, starts HTTP server + provider processes.
cmd/provider-musicbrainz/       Standalone gRPC server for MusicBrainz API
cmd/provider-deezer/            Standalone gRPC server for Deezer API
proto/provider/                 Protobuf service definition + generated Go code
internal/
  api/                          HTTP handlers (chi router), SPA serving
  activity/                     Activity log (separate SQLite: activity.db)
  cache/                        SQLite-backed cache (cache.db) with TTL
  config/                       Env-based config (CRATE_* vars)
  db/                           SQLite via modernc.org/sqlite, goose migrations, raw SQL queries
  migrations/                   Embedded .sql migration files
  models/                       Shared structs and status enums
  provider/
    manager.go                  Provider registry, search+enrichment, caching, health checks
    process.go                  Child process management for built-in providers
  services/
    slskd/                      slskd API client (Soulseek daemon)
    downloader/                 Background download queue processor (tick every 10s)
    scheduler/                  Background periodic jobs (new release detection, auto-queue, quality upgrades)
    organizer/                  Moves downloaded files into library, renames by convention
    tagger/                     ID3/FLAC metadata tagging + cover art embedding
    navidrome/                  Optional Navidrome integration (triggers library scan after download)
web/                            React frontend (Vite, Tailwind, React Query, React Router)
```

## Provider Architecture

Crate uses gRPC to communicate with music metadata providers. The Docker image ships with MusicBrainz and Deezer providers as child processes managed by the main crate binary.

```
crate (main process)
  ├── Provider Manager (routes requests, caches, enriches)
  │   ├── gRPC → provider-musicbrainz (port 50051, 1 req/s rate limit)
  │   ├── gRPC → provider-deezer (port 50052, 10 req/s rate limit)
  │   └── gRPC → any external provider
  ├── Scheduler (new release detection via entity's stored provider)
  ├── Downloader (slskd integration)
  └── HTTP API (provider-unaware frontend)
```

Key concepts:
- **Default provider**: used for search and browse (configurable in settings, default: musicbrainz). Users can switch providers on the fly from the search UI.
- **Provider tracking**: each entity (artist/album/track) stores which provider+ID it came from
- **Orphan detection**: entities whose provider is unhealthy show as "orphaned" in the UI
- **Relink**: any entity can be relinked to a different provider at any time
- **Cache**: separate SQLite DB (cache.db) with configurable TTL, clearable from settings

Provider config format: `CRATE_PROVIDERS=name:binary:port,name2:binary2:port2`
For external providers: `CRATE_PROVIDERS=spotify:external:192.168.1.10:50053`

## Data flow

1. User searches for an artist (via selected provider's gRPC API, switchable on the fly)
2. User watches an artist (full discography), album, or individual track
3. Watched items saved to SQLite with provider + provider_id
4. Scheduler (configurable interval, default 6h) checks each watched artist's provider for new releases
5. Downloader processes queue: search slskd → pick best file → download → organize → tag
6. Track status: `wanted` → `downloading` → `owned`

## Download Retry, Blacklist & Shadow Bans

- **No results**: immediate fail, no retry. Track stays "wanted" for the scheduler's next cycle.
- **Transfer failures** (rejected, errored, cancelled): the (username, filename) pair is blacklisted in `slskd_blacklist` table. Future searches skip that source.
- **Stale transfers**: state-aware timeouts detect stalled downloads. InProgress/Initializing = 5min, Queued = 30min, Requested = 10min. Active transfer stalls blacklist the file; queued/requested stalls trigger a shadow ban on the user.
- **Shadow bans (cooldowns)**: temporary per-user blocks stored in `user_cooldowns` table. Triggered by stale queued transfers or StartDownload failures (e.g. user offline). Duration is configurable via `shadow_ban_duration_minutes` setting (default 60min). Expired cooldowns are auto-purged on the scheduler's daily integrity tick. `scoreCandidates` skips cooled-down users entirely.
- **Retry backoff**: 5m → 15m → 30m → 1h. After 4 attempts (~2h cumulative), permanently fails. Track reverts to "wanted".
- **Blacklist is per-file-per-user**: a user can be blacklisted for one file but not others. Shadow bans are per-user (all files blocked during cooldown).
- **API management**: `GET/DELETE /api/blacklist/{id}`, `GET/DELETE /api/cooldowns/{id}` for viewing and removing entries. Also exposed in the Settings UI under "Blocked Sources".

## Pagination


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheOutdoorProgrammer/crate](https://github.com/TheOutdoorProgrammer/crate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-26 -->
