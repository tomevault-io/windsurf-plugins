---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Podsync is a Go-based service that converts YouTube, Vimeo, SoundCloud, and Twitch channels into podcast feeds. It downloads video/audio content and generates RSS feeds that can be consumed by podcast clients.

## Key Architecture Components

### Main Application (`cmd/podsync/`)
- **main.go**: Entry point with CLI argument parsing, signal handling, and service orchestration
- **config.go**: TOML configuration loading and validation with defaults

### Core Packages (`pkg/`)
- **builder/**: Media downloaders for different platforms (YouTube, Vimeo, SoundCloud, Twitch)
- **feed/**: RSS/podcast feed generation and management, OPML export, hooks, API key rotation
- **db/**: BadgerDB-based storage for metadata and state
- **fs/**: Storage abstraction supporting local filesystem and S3-compatible storage
- **model/**: Core data structures and domain models
- **ytdl/**: YouTube-dl wrapper for media downloading

### Services (`services/`)
- **update/**: Feed update orchestration, scheduling, episode filtering via matcher.go
- **web/**: HTTP server for serving podcast feeds, media files, and health checks
- **migrate/**: Filename migration tooling for transitioning to custom filename templates

### Key Dependencies
- youtube-dl/yt-dlp for media downloading
- BadgerDB for local storage
- go-toml for configuration
- robfig/cron for scheduling
- AWS SDK for S3 storage

## Episode Lifecycle

Understanding how episodes flow through the system:

### Discovery Phase
- Episodes are discovered during feed updates via platform APIs (YouTube Data API v3, Vimeo API, SoundCloud, Twitch)
- `updateFeed()` in `services/update/updater.go:99-154` queries the platform API
- New episodes are stored in BadgerDB with status `EpisodeNew`
- Episodes matching feed URL are identified by provider-specific parsing in `pkg/builder/`

### Download Phase
- `fetchEpisodes()` iterates episodes with status `EpisodeNew` or `EpisodeError`
- Episodes are filtered by match rules (title, description, duration, age) in `services/update/matcher.go:27-72`
- Only `page_size` episodes are queued per update cycle (default 50)
- Downloads happen to temp directory first, then copied to storage to prevent incomplete files
- On success: status set to `EpisodeDownloaded` with file size recorded
- On failure: status set to `EpisodeError`, retry attempted next cycle

### Cleanup Phase (Important!)
- `cleanup()` in `updater.go:373-441` runs AFTER each successful update cycle
- **Only triggered if `clean.keep_last` is configured** (global or per-feed)
- Keeps most recent N episodes by PubDate (descending order)
- Deleted episodes have status changed to `EpisodeCleaned` and title/description cleared
- **Files are deleted from storage but database records are retained**
- This is a soft-delete: episodes remain in the database forever

### Episode Removal from Database
- Episodes are removed from the database only if they:
  1. Are no longer returned by the platform API, AND
  2. Have status `EpisodeNew` (never downloaded)
- Episodes with status `EpisodeDownloaded` or `EpisodeCleaned` are NEVER removed from the database
- There is NO mechanism to compact or prune the database of old episode records

## Database Behavior (BadgerDB)

### Data Storage
- Uses versioned keyspace: `podsync/v1/`
- Key prefixes: `feed/{feedID}` for feeds, `episode/{feedID}/{episodeID}` for episodes
- Both use JSON serialization
- Records are append-only; deleted episodes remain with `EpisodeCleaned` status

### Database Growth and Limitations
- **Database grows indefinitely** as new episodes are discovered
- Deleted/cleaned episodes remain in DB forever (not physically removed)
- **No built-in compaction or garbage collection** mechanism
- No configuration option to prune old episode records
- `keep_last` only deletes files, not database records
- For very large feeds, database file size can grow significantly over time

### Configuration Options
```toml
[database]
dir = "/path/to/db"  # defaults to {config_dir}/db

[database.badger]
truncate = true      # enable value log truncation
file_io = true       # use file I/O instead of mmap
```

## Configuration Reference

### Feed Configuration (`[feeds.{ID}]`)
```toml
[feeds.my_feed]
url = "https://youtube.com/..."        # Required: platform URL
page_size = 50                         # Episodes to fetch per update (default 50)
update_period = "6h"                   # How often to check (default 6h)
cron_schedule = "0 */6 * * *"          # Cron expression (overrides update_period)
quality = "high"                       # "high" or "low" (default "high")
format = "video"                       # "audio", "video", or "custom"
max_height = 720                       # Maximum video height (720, 1080, 1440, etc.)
playlist_sort = "desc"                 # "asc" or "desc" for playlist ordering
filename_template = "{{id}}"           # Tokens: {{id}}, {{title}}, {{pub_date}}, {{feed_id}}
opml = true                            # Include in OPML export
private_feed = false                   # Don't index by podcast aggregators

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mxpv/podsync](https://github.com/mxpv/podsync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
