---
trigger: always_on
description: Extract health, activity, workout, and performance data from Garmin Connect. Use when users need Garmin fitness data including sleep, HRV, weight, stress, activities, workouts, and performance metrics.
---


# garmin-cli

Extract health, activity, workout, and performance data from Garmin Connect.

## Prerequisites

- Python 3.10+
- Install: `pip install .` (from repo root)
- Authentication: run `garmin-cli login`, or use a saved session at `~/.garminconnect`, or set env vars `GARMIN_EMAIL` / `GARMIN_PASSWORD`

## Agent Usage

Always use `--json` for machine-readable output. The tool returns a JSON envelope on stdout with exit code 0 (success) or 1 (error).

### Success envelope

```json
{
  "ok": true,
  "command": "<group> <subcommand>",
  "date_range": null,
  "count": 7,
  "data": [...]
}
```

`date_range` is `null` for commands that are not scoped to a date range. When a command is date-scoped, it is an object with `from` and `to` ISO dates.

### Error envelope

```json
{
  "ok": false,
  "command": "<group> <subcommand>",
  "error": "Human-readable message",
  "error_code": "RATE_LIMITED"
}
```

### Error codes

| Code | Meaning | Retry? |
|------|---------|--------|
| `AUTH_MISSING` | No credentials or session found | No -- configure auth |
| `AUTH_FAILED` | Credentials rejected | No -- fix credentials |
| `NOT_FOUND` | Endpoint returned 404 | No |
| `RATE_LIMITED` | 429 after 3 retries | Yes -- wait and retry |
| `SERVER_ERROR` | 5xx after 3 retries | Yes -- wait and retry |
| `INVALID_INPUT` | Bad arguments or conflicting options | No -- fix arguments |
| `INTERNAL_ERROR` | Unexpected error, including uncategorized connection/timeout failures | Maybe |

## Commands

### Date range options (health range commands + workout calendar)

| Option | Effect | Example |
|--------|--------|---------|
| `--date YYYY-MM-DD` | Single day | `--date 2026-03-11` |
| `--days N` | Past N days (inclusive of today) | `--days 7` |
| `--from YYYY-MM-DD --to YYYY-MM-DD` | Explicit range (both inclusive) | `--from 2026-03-01 --to 2026-03-07` |
| `--ahead N` | Next N days (workout calendar only) | `--ahead 7` |
| *(none)* | Defaults to today | |

Max range: 90 days. Conflicting options produce `INVALID_INPUT`.

---

### Health

All health commands except `health status` accept the shared date range options. `health status` only accepts `--date` and defaults to today.

```bash
# Sleep -- fields: date, duration_hours, deep_min, light_min, rem_min, awake_min, score
garmin-cli --json health sleep --days 7

# HRV -- fields: date, weekly_avg, last_night, status
garmin-cli --json health hrv --date 2026-03-11

# Weight -- fields: date, weight_kg, bmi, body_fat_pct
garmin-cli --json health weight --days 30

# Body battery -- fields: date, start_level, end_level
garmin-cli --json health body-battery --days 7

# Stress -- fields: date, avg_stress, max_stress
garmin-cli --json health stress --days 7

# SpO2 -- fields: date, avg_spo2, lowest_spo2
garmin-cli --json health spo2 --days 7

# Resting heart rate -- fields: date, resting_hr
garmin-cli --json health resting-hr --days 7

# Training readiness -- fields: date, score, level
garmin-cli --json health readiness --days 7

# Training status (single day only) -- fields: date, training_status, load_type
garmin-cli --json health status --date 2026-03-11

# Steps -- fields: date, total_steps, total_distance, step_goal
garmin-cli --json health steps --days 7

# Daily summary -- fields: date, total_steps, distance_km, calories, floors, moderate_intensity_minutes, vigorous_intensity_minutes, resting_hr
# Note: one API call per day — large ranges may be slow
garmin-cli --json health daily-summary --days 7

# Intensity minutes -- fields: date, moderate_value, vigorous_value, weekly_goal
garmin-cli --json health intensity-minutes --days 7
```

### Activities

```bash
# List recent activities -- fields: id, date, name, type, distance_km, duration_min, avg_hr
garmin-cli --json activity list --limit 10
garmin-cli --json activity list --limit 10 --type running
garmin-cli --json activity list --limit 10 --search "morning run"
garmin-cli --json activity list --from 2026-03-01 --to 2026-03-31
garmin-cli --json activity list --days 7

# Get a single activity by ID -- same fields as list
garmin-cli --json activity get 12345678901

# Sport-aware detail. Cycling rides surface power suite (avg/max/normalized,
# TSS, IF); running activities surface cadence_spm, GCT, vertical
# oscillation/ratio, stride length, training effect; pool swims surface
# SWOLF, total strokes, average stroke rate, distance per stroke. JSON
# uses a stable union schema -- every key present (null for sport-
# inapplicable). Includes a top-level `unavailable` array when any
# registry-known metric is not produced.
garmin-cli --json activity get 12345678901 --detail

# Detail + lap data in one envelope (--laps appends `laps` array).
# Pool-swim activities auto-route to per-pool-length rows.
garmin-cli --json activity get 12345678901 --detail --laps

# Lap-by-lap data (run/bike) or per-pool-length data (lap_swimming)
garmin-cli --json activity laps 12345678901

# HR time-in-zone breakdown -- fields: zone, zone_low_bpm, zone_high_bpm, seconds_in_zone, minutes_in_zone
garmin-cli --json activity zones 12345678901


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ching-kuo/garmin-cli](https://github.com/ching-kuo/garmin-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
