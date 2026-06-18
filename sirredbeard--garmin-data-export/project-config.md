---
trigger: always_on
description: This file is for AI agents working on or consuming output from this project.
---

# AGENTS.md -- AI context for garmin-data-export

This file is for AI agents working on or consuming output from this project.
For human setup instructions, see [README.md](README.md).

## What this project does

A single Python script (`garmin_export.py`) downloads all available health
and fitness data from a user's Garmin Connect account and writes it as one
plain text file with raw JSON data blocks. The output is designed for LLM
consumption -- every API response is dumped as complete, unfiltered JSON.
No markdown formatting is used; plain text headers separate sections.

This format was specifically chosen for compatibility with NotebookLM and
other LLM tools. Research found that .md files have known parsing bugs in
NotebookLM's RAG indexer, and content inside code fences (```json) gets
skipped. Plain .txt with raw JSON avoids both issues.

No official Garmin API key exists for personal use. The script authenticates
through Garmin's SSO (same flow as the website) via the
[python-garminconnect](https://github.com/cyberjunky/python-garminconnect)
library, which wraps [garth](https://github.com/matin/garth) for OAuth.

## Output file structure

Each export produces a single file: `export/garmin_export_YYYY-MM-DD_HHMMSS.txt`
Update exports use: `export/garmin_export_YYYY-MM-DD_HHMMSS_update.txt`

The output uses plain text section headers and raw JSON (no markdown headings,
no code fences, no bold/italic). This is intentional -- NotebookLM's RAG
indexer has bugs with .md files and skips content inside code fences.

```
Garmin Connect Data Export            -- title + metadata (date range, export time)

Table of Contents                     -- numbered list with section names and descriptions;
                                         notes that all data is raw JSON
Profile                               -- section-level cache (fetched once, cached forever)
Daily Health                          -- per-day: 13 endpoints fetched concurrently (4 threads)
  2026-03-22                          -- one per day, newest first
    Steps / Heart Rate / Sleep        -- each sub-section has a plain text title + raw JSON
Activities                            -- per-activity cache; list then detail per item
  Activity {id}                       -- summary, splits, zones, weather, time-series
Body Composition                      -- chunked yearly API calls, section cache
Training Metrics                      -- VO2max, FTP, hill/endurance scores, etc.
Goals and Records                     -- PRs, badges, goals
Trends                                -- weekly aggregates, daily steps, floors, progress
Golf                                  -- list then scorecard + shots per round
Gear                                  -- needs userProfileNumber; list then stats per item
Training Plans                        -- list then detail per plan
Workouts                              -- list then detail per workout
Hydration                             -- per-day, fetched concurrently (4 threads)
Nutrition                             -- per-day (3 calls/day), fetched concurrently
Women's Health                        -- chunked date range + pregnancy summary

Errors During Export                  -- only if any sections failed
```

Empty sections contain: "No data available."

## Architecture

### Single file, three classes

| Class | Purpose |
|-------|---------|
| `RateLimiter` | Thread-safe adaptive pacer. Starts at 0.15s delay, doubles on 429, decays on success. Forced 2s pause every 250 calls. |
| `ExportCache` | Persistent JSON file cache under `export/.cache/`. Three namespaces: `daily/` (per-day), `activities/` (per-activity), `sections/` (whole-section blobs). Never invalidates -- only `--no-cache` or deleting `.cache/` forces re-fetch. |
| `GarminExporter` | Orchestrator. Authenticates, iterates the sections list, writes plain text output, handles Ctrl-C gracefully (saves partial export). |

### Concurrency model

- Daily health: `ThreadPoolExecutor(max_workers=4)` -- 13 endpoints per day fetched in parallel
- Hydration: 4 days fetched concurrently (1 API call per day)
- Nutrition: 4 days concurrently (3 API calls per day)
- Activities: sequential (each activity has ~8 detail calls)
- `RateLimiter.wait()` uses a `threading.Lock` so concurrent threads are still paced

### Caching strategy

Cache is **permanent**. Once a day/activity/section is cached, it is never
re-fetched unless the user passes `--no-cache` or deletes the cache directory.

| Cache type | Key | Location |
|------------|-----|----------|
| Daily health | `YYYY-MM-DD` | `export/.cache/daily/YYYY-MM-DD.json` |
| Hydration | `hydration_YYYY-MM-DD` | `export/.cache/daily/hydration_YYYY-MM-DD.json` |
| Nutrition | `nutrition_YYYY-MM-DD` | `export/.cache/daily/nutrition_YYYY-MM-DD.json` |
| Activity | `{activityId}` | `export/.cache/activities/{activityId}.json` |
| Section | `{name}` | `export/.cache/sections/{name}.json` |

On re-run, only uncached items are fetched. This makes interrupted `--all`
exports fully resumable -- just run the same command again.

### Chunked date-range calls

Several Garmin endpoints reject date ranges longer than about one year with
HTTP 400. The `_chunked_date_call()` helper breaks ranges into 365-day

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sirredbeard/garmin-data-export](https://github.com/sirredbeard/garmin-data-export) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
