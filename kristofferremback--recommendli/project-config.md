---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Recommendli is a Spotify recommendation service that analyzes playlists and listening habits to generate personalized music recommendations. It filters and ranks tracks from Discovery Weekly and Release Radar playlists, excluding songs already in the user's library.

**Tech Stack**: Go backend with vanilla JavaScript frontend, SQLite for persistence, Spotify API integration.

## Development Commands

### Running the Application

```bash
# Run locally (requires .env file with Spotify credentials)
make dev

# Build executable
make build

# Run with Docker Compose
docker-compose up --build
```

### Environment Variables

Required environment variables (configure in `.env` for local development):

- `SPOTIFY_ID` - Spotify client ID
- `SPOTIFY_SECRET` - Spotify client secret
- `SPOTIFY_REDIRECT_HOST` - OAuth redirect host (default: `http://localhost:9999`)
- `ADDR` - Server address (default: `0.0.0.0:9999`)
- `LOG_LEVEL` - Logging level (default: `info`)
- `FILE_CACHE_BASE_DIR` - Cache directory (default: `/tmp/recommendli`)
- `SQLITE_DB_PATH` - SQLite database path (default: `/tmp/recommendli.sqlite`)

### Spotify OAuth Configuration

**Important**: Spotify requires explicit IP addresses for local redirect URIs, NOT `localhost`.

- ✅ Use: `http://127.0.0.1:9999/recommendations/v1/spotify/auth/callback`
- ❌ Don't use: `http://localhost:9999/recommendations/v1/spotify/auth/callback`

**Why**: The hostname `localhost` can be hijacked via hosts file manipulation (`/etc/hosts`), allowing an attacker to intercept OAuth authorization codes. IP addresses like `127.0.0.1` (IPv4) or `::1` (IPv6) are hardcoded in the OS and cannot be overridden, preventing this attack vector.

**Local development setup**:
1. In Spotify Developer Dashboard, register redirect URI: `http://127.0.0.1:9999/recommendations/v1/spotify/auth/callback`
2. Set `SPOTIFY_REDIRECT_HOST=http://127.0.0.1:9999` in `.env`
3. Access the app at `http://127.0.0.1:9999` (not `localhost`)

**Production setup**:
- Register redirect URI: `https://recommendli.remback.se/recommendations/v1/spotify/auth/callback`
- Set `SPOTIFY_REDIRECT_HOST=https://recommendli.remback.se` in production `.env`

### Build Details

- CGO is **enabled** for SQLite support (`CGO_ENABLED=1`)
- Production Dockerfile disables CGO (`CGO_ENABLED=0`) and uses distroless base image
- Built executable location: `./build/main`

## Architecture

### Core Concepts

**Track Index**: SQLite-backed index that maps tracks to playlists. The system syncs playlist snapshots, detects changes (added/changed/removed playlists), and maintains a normalized view of user's library. This enables fast lookups to check if a track already exists in any user playlist.

**Singleflight Locking**: The `pkg/singleflight` package provides distributed locking for expensive operations like syncing the track index. When multiple requests try to sync simultaneously, only one proceeds while others wait and poll. Locks auto-refresh using a ticker to prevent expiration during long operations.

**Spotify Adaptor**: Wraps the Spotify API client with caching (via KeyValueStore) and pagination helpers. Split across multiple files:
- `spotifyadaptor.go` - Core adaptor, tracks, current user
- `spotifyadaptor_auth.go` - OAuth flow handlers
- `spotifyadaptor_albums.go` - Album operations
- `spotifyadaptor_playlists.go` - Playlist operations

**Service Layer**: Business logic lives in `internal/recommendations/recommendations.go`. The service uses dependency injection via factories to compose SpotifyProvider, KeyValueStore, UserPreferences, and TrackIndex.

### Key Components

#### `internal/recommendations/`
- `recommendations.go` - Core service implementing recommendation logic
- `recommendations_lib.go` - Helper functions for track scoring and filtering
- `http_handler.go` - HTTP handlers using chi router
- `trackindex.go` - Interface for track-to-playlist index
- `keyvaluestore.go` - Generic key-value persistence interface
- `spotifyadaptor*.go` - Spotify API client wrapper with caching

#### `internal/sqlite/`
- `db.go` - Wrapper around sqlx.DB with RWMutex for safe concurrent access
- `trackindex.go` - SQLite implementation of TrackIndex interface
- `keyvaluestore.go` - SQLite-backed key-value store
- `locker.go` - SQLite-backed distributed lock for singleflight

#### `pkg/`
Reusable utilities:
- `singleflight/` - Distributed locking with auto-refresh
- `slogutil/` - Structured logging helpers (slog wrappers)
- `migrations/` - Database migration runner (uses golang-migrate)
- `paginator/` - Pagination helpers
- `srv/` - HTTP utilities (JSON responses, 404 redirector)
- `ctxhelper/` - Context utilities
- `sortby/` - Sorting helpers
- `spotifyutil/` - Spotify-specific utilities

### Database

Migrations are in `./migrations/` and run automatically on startup via `migrations.Up()` in `main.go`.

Key tables:
- `keyvaluestore` - Generic key-value storage
- `trackindex_playlists` - Stores playlist snapshots per user
- `trackindex_playlist_tracks` - Normalized track-to-playlist mapping
- `singleflight_locker` - Distributed lock state

### Request Flow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kristofferremback) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
