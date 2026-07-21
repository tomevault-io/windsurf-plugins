---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) and Codex (Codex.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) and Codex (Codex.ai/code) when working with code in this repository.

## What this project is

Tubeless is a self-hosted Phoenix/Elixir web app that wraps `yt-dlp` to automatically download YouTube content (channels, playlists) on a schedule. It uses SQLite (via `ecto_sqlite3`), Oban for background jobs, and Phoenix LiveView for the UI. Deployed as a single Docker container with no external dependencies.

## Commands

> **On macOS, run tests through Docker, not natively.** The suite needs Linux-only
> binaries (SQLean `.so`, yt-dlp/ffmpeg/Deno/Apprise) that aren't available on the
> host, so `mix test` / `mix check` will not work directly. Use the two wrapper
> scripts below — they run inside the pinned ci-base image and share a warm build
> cache:
>
> - `tooling/test.sh [args…]` — **fast iteration loop.** Passes everything through
>   to `mix test`, skips the non-test checks and asset builds. This is what you run
>   after editing code.
>   - `tooling/test.sh test/path/to/file_test.exs` — one file
>   - `tooling/test.sh test/path/to/file_test.exs:42` — one test by line
>   - `tooling/test.sh --failed` / `--stale` — re-run last failures / affected tests
>   - `tooling/test.sh` (no args) — whole suite
>   - `tooling/test.sh --clean …` — wipe the cached volumes first
>   - `tooling/test.sh --shell` — drop into a shell in the container
> - `tooling/lint_test.sh` — **pre-commit gate.** Full `mix check` (compiler, credo, sobelow, prettier, full ExUnit) inside the same image. Slower; run once before committing, not while iterating. Shares volumes with `test.sh`.
>
> The bare `mix …` commands below are the underlying tasks these scripts run; on a
> Linux dev box (or inside `--shell`) you can call them directly.

```bash
# Initial setup
mix setup               # deps.get + DB create/migrate/seed + asset setup/build

# Development server (inside Docker or with deps installed)
iex -S mix phx.server  # starts on port 4008 in dev

# Tests
mix test                          # run all tests (creates/migrates DB automatically)
mix test test/path/to/file_test.exs          # single file
mix test test/path/to/file_test.exs:42       # single test by line number

# Full quality check (what CI runs)
mix check                         # formatter + compiler + sobelow + prettier + ex_unit

# Individual checks
mix credo                         # Elixir code style
mix sobelow --config              # security scan
yarn run lint:check               # Prettier (JS/CSS)
yarn run lint:fix                 # Prettier auto-fix

# Database
mix ecto.migrate                  # run migrations (also regenerates priv/repo/erd.png)
mix ecto.rollback                 # rollback one migration
mix ecto.reset                    # drop + recreate + migrate + seed

# Assets
mix assets.build                  # compile Tailwind + esbuild (dev)
mix assets.deploy                 # minified build + digest for production

# Versioning
mix version.bump                  # runs tooling/version_bump.sh
```

## Architecture

## Codebase

- Project layout is stored in @CODEBASE.md

### Core domain model

Two top-level entities drive everything:

- **`Source`** (`lib/pinchflat/sources/`) — a YouTube channel or playlist the user wants to track. Has a `MediaProfile` that defines download rules.
- **`MediaItem`** (`lib/pinchflat/media/`) — a single video/audio item belonging to a Source. Tracks download state, file paths, metadata.
- **`MediaProfile`** (`lib/pinchflat/profiles/`) — reusable settings for how to download (format, quality, naming, Shorts/livestream rules, SponsorBlock, etc.).

### Background job system

All async work is done through **Oban** jobs. There's a thin wrapper called **`Task`** (`lib/pinchflat/tasks/`) that links an `Oban.Job` to either a `Source` or `MediaItem`. Use `Tasks.create_job_with_task/2` when scheduling work — it handles deduplication and the task record atomically.

Key workers:

- `FastIndexingWorker` (`lib/pinchflat/fast_indexing/`) — polls YouTube RSS feeds to detect new videos quickly without hitting the API
- `MediaCollectionIndexingWorker` (`lib/pinchflat/slow_indexing/`) — full yt-dlp metadata fetch for a Source (slow indexing)
- `MediaDownloadWorker` (`lib/pinchflat/downloading/`) — downloads a single MediaItem via yt-dlp
- `MediaQualityUpgradeWorker` (`lib/pinchflat/downloading/`) — re-downloads media after a configured delay to get better quality
- `MediaRetentionWorker` (`lib/pinchflat/downloading/`) — deletes old media per retention settings
- `SourceMetadataStorageWorker` (`lib/pinchflat/metadata/`) — fetches and stores source-level metadata (images, NFO, etc.)
- `FileSyncingWorker` (`lib/pinchflat/media/`) — reconciles MediaItem records with files on disk
- `SourceDeletionWorker` (`lib/pinchflat/sources/`) — cascades deletion of a Source and all its media
- `MediaProfileDeletionWorker` (`lib/pinchflat/profiles/`) — cascades deletion of a MediaProfile

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CommunityMaintained/tubeless](https://github.com/CommunityMaintained/tubeless) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
