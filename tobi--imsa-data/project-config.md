---
trigger: always_on
description: The IMSA database (`output/imsa.duckdb`) provides comprehensive racing data from
---

# IMSA Data Agent Guide

The IMSA database (`output/imsa.duckdb`) provides comprehensive racing data from
2021-2025. This guide documents the schema so downstream agents (human or
automated) can reason about the data without reverse-engineering the SQL pipeline.

## Quick Start

**Start with the `seasons` table** for a high-level overview of all sessions,
events, and races. This table provides aggregated statistics that help you
understand the scope of the data before diving into individual laps.

```sql
SELECT * FROM seasons WHERE session = 'race' ORDER BY date DESC LIMIT 10;
```

---

## The `seasons` View

The `seasons` view provides a session-level summary of every practice, qualifying,
and race session across all years. Use this table to identify which sessions exist,
when they occurred, and high-level characteristics like total laps and weather
conditions.

### Columns

| Column | Type | Description |
| --- | --- | --- |
| `date` | DATE | Event start date (first session of the event). |
| `session_id` | BIGINT | Unique identifier for this session. Use as join key with `laps` table. |
| `season` | VARCHAR | Season year (`2021`–`2025`). |
| `event` | VARCHAR | Canonical venue name (e.g., `Road America`, `Sebring`). |
| `session` | VARCHAR | Session type (`race`, `practice-1`, `practice-2`, `qualifying`, etc.). |
| `cars` | BIGINT | Count of distinct cars that participated in this session. |
| `drivers` | BIGINT | Count of distinct drivers who competed. |
| `classes` | VARCHAR | Comma-separated list of classes that participated (e.g., `GTP, LMP2, GTD`). |
| `session_start` | TIMESTAMP | Actual session start timestamp. |
| `session_end` | TIMESTAMP | Session end timestamp (computed from max session_time). |
| `total_laps` | INTEGER | Total number of laps completed across all cars. |
| `rain_laps` | BIGINT | Count of laps completed during rain conditions. |
| `flags` | VARCHAR | Distinct flag conditions during the session, excluding green flags (e.g., `FCY, RED`). |

### Usage Tips

- Filter by `session = 'race'` to focus on race sessions only
- Use `session_id` as a join key when connecting to the `laps` table
- Check `rain_laps` to quickly identify wet sessions
- The `flags` column shows non-green flag conditions; NULL means green flags only

---

## The `laps` Table

The `laps` table is the canonical surface for detailed lap-by-lap analysis,
matching every recorded lap with driver, team, and weather context.

## Row Semantics

Each row represents a single completed lap for a given car during a specific
session. Driver, team, and license values are taken from the authoritative
entry list (`event_drivers`) whenever available; the raw lap CSV values are
used only as fallbacks. Weather fields reflect the most recent observation at
or before the lap's `session_time`.

## Columns

| Column | Type | Description |
| --- | --- | --- |
| `start_date` | TIMESTAMP | Scheduled start timestamp for the session, parsed from the IMSA file name. Used in combination with `year`, `event`, and `session` to disambiguate double-header weekends (e.g., two Watkins Glen races in 2021). |
| `year` | VARCHAR | Season year (`2021`–`2025` currently). Purely textual so it can coexist with historical prefabricated datasets. |
| `event` | VARCHAR | Canonical venue name from `clean_event_name` (e.g., `Road America`, `Sebring`). Normalization prevents small spelling changes from splitting events. |
| `session` | VARCHAR | IMSA session label such as `race`, `practice-2`, `qualifying`. Mirrors the official results naming so you can correlate back to the PDFs. |
| `session_id` | BIGINT | Deterministic identifier created with `DENSE_RANK` over `(year, event, session, start_date)`. This means every unique year/event/session/start_date combination maps to exactly one `session_id`, while all laps within that session share it. Reliable for partitioning or join keys across derived tables. |
| `session_time` | DECIMAL(10,3) | Seconds elapsed in the session at the moment the car finished the lap. Display as `MM:SS.mmm` for readability if you surface it to humans. |
| `clock_time` | DECIMAL(10,3) | Wall-clock time of lap completion relative to the session start; mirrors IMSA’s “Hour” column in decimal seconds. Prefer formatting as `HH:MM:SS.mmm` in output. |
| `session_time_lap_number` | INTEGER | Virtual lap counter based on session timing. It increments whenever any car (usually the leader) completes another lap, so heavily delayed cars line up with the field’s current progress. |
| `car` | VARCHAR | Car number exactly as reported, including leading zeros (`'01'`, `'007'`). Keeping it textual avoids collisions between `01` and `1`. |
| `class` | VARCHAR | Competition class (`GTP`, `LMP2`, `GTD`, etc.). Derived from the lap feed. |
| `driver_name` | VARCHAR | Driver name for display and filtering. Prefers the entry list canonical name when available, otherwise falls back to the normalized lap CSV value. Use this for most queries. |
| `driver_id` | VARCHAR | Stable identifier for the driver across all sessions and name variations (e.g., "mathias beche_2025"). Use this for aggregating laps across spelling variants like "Mathias BECHE" vs "Mathias Beche". |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tobi/imsa_data](https://github.com/tobi/imsa_data) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
