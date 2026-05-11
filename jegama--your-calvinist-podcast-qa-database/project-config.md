---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FastAPI application that extracts timestamped Q&A content from YourCalvinist Podcast YouTube videos, classifies them using Gemini AI, and serves searchable endpoints. Designed for serverless deployment on Vercel with Neon PostgreSQL.

The app now also exposes an MCP server for LLM clients and a human-facing `POST /v1/ask` endpoint for archive Q&A.

**Live API**: https://keithfoskey.calvinistparrot.com

## Development Commands

### Start Development Server
```bash
# Install dependencies
pip install -r requirements.txt

# Start API with auto-reload
uvicorn app.main:app --reload
```

### Local Video Processing
```bash
# Process all videos from file
python -m app.cli.backfill --input playlist_videos.txt

# Process only unprocessed videos
python -m app.cli.backfill --input playlist_videos.txt --skip-processed

# Skip AI classification (faster, for testing)
python -m app.cli.backfill --input playlist_videos.txt --skip-classification

# Dry run (don't save to database)
python -m app.cli.backfill --input playlist_videos.txt --dry-run

# Process limited number with delay
python -m app.cli.backfill --input playlist_videos.txt --limit 5 --delay 2

# Re-process all videos from stored transcripts (no YouTube API calls)
python -m app.cli.backfill --from-stored

# Re-process with limit
python -m app.cli.backfill --from-stored --limit 5 --delay 2
```

### Manual Timestamp Ingestion
```bash
# For videos with manually extracted timestamps
python -m app.cli.ingest_manual_timestamps
```

## Architecture

### Core Pipeline Flow (`app/ingest/pipeline.py`)

The `process_video()` function is the single source of truth for video processing.
`reprocess_from_stored_transcript()` re-processes from stored DB transcripts without YouTube API calls (for re-classification).

1. **Extract video ID** via `app/youtube/ids.py` - handles both IDs and URLs
2. **Fetch metadata** via YouTube Data API (`app/youtube/metadata.py`)
3. **Parse timestamps** from description (`app/qa/timestamp_parser.py`) - extracts questions with timestamps
4. **Fetch transcript** segments (`app/youtube/transcripts.py`) - gets timestamped text
5. **Store raw transcript** in `transcripts` table as JSONB for re-processing without YouTube API hits
6. **Slice answers** by timestamp windows (`app/qa/answer_slicer.py`) - start-to-next-start windows
7. **Classify with Gemini** (`app/qa/classify.py`) - optional, uses `categories.json` schema
8. **Persist to database** via CRUD operations in `app/db/crud.py`

### Database Schema

**Key tables** (see `app/db/models.py`):
- `videos` - YouTube video metadata and processing status
- `qa_items` - Question-answer pairs with timestamps (unique on `video_id, timestamp_seconds`), includes `passages TEXT[]` for cited Bible passages
- `tags` - Tag names (many-to-many with qa_items)
- `transcripts` - Raw JSONB transcript segments + full text (kept separate for performance)
- `ingest_jobs` - Lightweight queue for video processing (status: pending → processing → done/failed)

**Full-text search**:
- `qa_items.search_tsv` column (tsvector) - declared in ORM and populated by Postgres trigger
- Must apply DDL from `plan.md` Section 3 before search works
- Trigger auto-updates `search_tsv` on insert/update using `to_tsvector('english', question || answer)`

**Schema migrations**:
- Ad-hoc SQL lives in `migrations/`; apply manually against Neon with `psql "$DATABASE_URL" -f migrations/<file>.sql`
- `001_add_passages_column.sql` — adds `qa_items.passages TEXT[]`. Apply before deploying code that writes to this column.

### Database Access Patterns

- **Request scope**: Use `get_db` dependency from `app/dependencies.py` in FastAPI routes
- **Internal jobs**: Use `get_session()` context manager from `app/db/engine.py`
- **Never** keep sessions open across requests - serverless constraint
- CRUD helpers in `app/db/crud.py` handle upserts, job locking, and tag creation

### Authentication

Two auth methods handled in `app/dependencies.py`:
- `X-API-Key: {ADMIN_API_KEY}` - for manual ingestion triggers
- `Authorization: Bearer {CRON_SECRET}` - for Vercel cron jobs

Public GET endpoints require no auth.

### Serverless Constraints (Vercel)

- **Bounded execution**: Keep handlers short (<30s), use queue for long operations
- **No long-lived connections**: Close DB sessions immediately
- **Side-effect free**: GET handlers should be idempotent
- **Cron configuration**: `vercel.json` defines two daily cron jobs at 07:00 and 07:05 UTC

### Queue System

Lightweight queue using `ingest_jobs` table:
- `enqueue_video()` - adds job if not already queued
- `get_and_lock_pending_job()` - atomic SELECT FOR UPDATE to claim job
- Failed jobs retry up to 3 times (configurable in cron logic)
- Check `check_recent_videos.ipynb` for requeue utilities

### Ingestion Endpoints (`app/routers/ingest.py`)

Protected by API key or cron secret:
- `GET/POST /v1/ingest/check` - scans playlist for new videos, enqueues unseen ones
- `GET/POST /v1/ingest/run-one` - processes one pending job
- `GET/POST /v1/ingest/run-batch?max_jobs=N` - loops up to N jobs (Vercel cron uses max_jobs=5)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jegama/Your-Calvinist-Podcast-QA-Database](https://github.com/Jegama/Your-Calvinist-Podcast-QA-Database) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
