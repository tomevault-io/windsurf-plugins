---
trigger: always_on
description: Artillery is a Flask-based UI for managing `gallery-dl` downloads with scheduling, task isolation, and a media wall dashboard. This guide covers critical architectural patterns and workflows.
---

# Artillery Copilot Instructions

Artillery is a Flask-based UI for managing `gallery-dl` downloads with scheduling, task isolation, and a media wall dashboard. This guide covers critical architectural patterns and workflows.

## Architecture Overview

**Three core processes:**
1. **Flask web server** (`app.py`) - REST API + UI for task management, config editing, media wall
2. **Cron scheduler** (`scheduler.py`) - Runs every minute, checks `cron.txt` files, spawns background task runners
3. **Media wall indexer** (`mediawall_index.py`) - Scans `gallery-dl` logs to catalog downloads into SQLite, caches thumbnails

**Task isolation model:**
- Each task lives in `/tasks/<slug>/` with files: `urls.txt`, `command.txt`, `cron.txt`, `logs.txt`, `name.txt`, `lock`, `paused`
- Lock file (`lock`) indicates task is running; `paused` file means task won't run via cron (manual runs still allowed)
- Gallery-dl config is shared globally at `/config/gallery-dl.conf` (editable via UI)

**Data flows:**
- UI → create task → writes `/tasks/slug/{name.txt,urls.txt,command.txt,cron.txt}`
- Cron triggers → `scheduler.py` detects matching cron.txt → spawns `run_task_background()` thread
- Task execution → gallery-dl outputs to `/downloads/<site>/<artist>/<file>`
- Media wall → parses task logs for file paths → indexes into SQLite → copies latest 100 to `/config/media_wall/` cache

## Key Implementation Patterns

**File-based state (no DB for tasks):**
- Task metadata stored as text files, not database. Use `read_text(path)` and `write_text(path, content)` helpers
- Slugs are derived from task names via `slugify()` (lowercase, hyphens, alphanumeric only)
- Always check for `lock` and `paused` files before state changes

**Subprocess execution (`run_task_background`):**
- Runs in daemon thread; sets `GALLERY_DL_CONFIG` env var pointing to shared config
- Command is parsed with `shlex.split()` to handle quoted args; run from task directory
- All stdout/stderr appended to `logs.txt` with timestamps and exit codes
- **Lock file removed in finally block** to ensure cleanup even on error

**Media wall indexing:**
- Two separate modules: `app.py` has inline SQLite logic, `mediawall_index.py` is standalone for future background workers
- DB schema: `media` table (path, ext, task, first_seen, last_seen, seen_count) + `task_offsets` table (tracks log file offset per task)
- Log ingestion uses file offset to only parse new lines; on task completion, auto-ingests and refreshes cache if throttle allows
- Cache refresh throttled by `MEDIA_WALL_MIN_REFRESH_SECONDS` (default 300s) to avoid copying 100 files per task in rapid succession

**Threading & concurrency:**
- Background task runs in daemon thread (`threading.Thread(..., daemon=True)`)
- Media wall refresh guarded by `MEDIA_WALL_REFRESH_LOCK` to prevent concurrent cache copies
- Cron scheduler runs as separate process (via crontab) every minute; checks for `lock` and `paused` before execution

**Flask routing patterns:**
- `/` (home) - renders media wall dashboard (3 rows of cached images, conditional on `MEDIA_WALL_ENABLED`)
- `/tasks` - GET lists all tasks, POST creates new task
- `/tasks/<slug>/action` - POST for run/pause/delete actions
- `/tasks/<slug>/logs` - GET returns JSON with task log content (used by real-time output viewer)
- `/config` - GET shows editor + media wall controls, POST saves gallery-dl.conf or handles media wall actions
- `/mediawall/toggle` - POST toggles media wall enabled/disabled (accessible via button on config page)
- `/mediawall/refresh` - POST refreshes wall cache
- `/mediawall/seed` - POST rebuilds media index then refreshes cache (accessible via "Refresh media wall" button on config page)
- `/mediawall/status` - GET returns media wall status
- `/mediawall/api/cache_index` - returns paginated JSON of cached media
- `/wall/<filename>` - serves cached media files

**Real-time task output viewer:**
- Located on `/tasks` page as a collapsible "Output" card panel below the task table
- `/tasks/<slug>/logs` endpoint returns JSON: `{"slug": slug, "content": log_text}`
- JavaScript polls every 1 second for live log updates with auto-scroll
- Log level pattern parsing via `parseLogColors()` function maps log level tags to CSS classes:
  - `[warning]` → `.log-warning`
  - `[error]` → `.log-error`
  - `[success]` → `.log-success`
  - `[info]` → `.log-info`
  - `[debug]` → `.log-debug`
- Collapsible output panel with task selector dropdown (Show/Hide button)
- Auto-refresh stops when panel is hidden to reduce polling overhead
- Auto-scroll pauses when user manually scrolls up in the log container

## Critical Developer Workflows

**Local development:**
```bash
pip install -r requirements.txt
export TASKS_DIR=/tmp/tasks CONFIG_DIR=/tmp/config DOWNLOADS_DIR=/tmp/downloads
python app.py  # Flask dev server on :5000
```

**Debug mode:**
- `ARTILLERY_LOG_LEVEL=DEBUG` - verbose logging
- `ARTILLERY_DEBUG_REQUESTS=1` - log request timing
- `ARTILLERY_DEBUG_FS=1` - log filesystem operation timing
- `ARTILLERY_HANG_DUMP_SECONDS=30` - dump thread stacks after 30s (signals SIGUSR1)

**Docker build/run:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ObviousViking/Artillery](https://github.com/ObviousViking/Artillery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
