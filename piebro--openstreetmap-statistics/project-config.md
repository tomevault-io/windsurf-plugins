---
trigger: always_on
description: The project analyzes OpenStreetMap data through preprocessed parquet files that provide detailed information about changesets, their comments, notes, and note comments.
---

# Creating New Statistics

The project analyzes OpenStreetMap data through preprocessed parquet files that provide detailed information about changesets, their comments, notes, and note comments.

## Available Datasets

The project includes 4 different datasets:

1. **changeset_data** - Main dataset containing all changesets with enriched metadata
   - Location: `../changeset_data/year=*/month=*/*.parquet`
   - Partitioned by year and month
   - Contains enriched columns like `created_by`, `device_type`, `imagery_used`, etc.

2. **changeset_comments_data** - Comments on changesets (changeset discussions)
   - Location: `../changeset_comments_data/*.parquet`
   - Not partitioned, stored as flat parquet files
   - Join with changeset_data using `changeset_id`

3. **notes_data** - Notes on the map
   - Location: `../notes_data/*.parquet`
   - Not partitioned, stored as flat parquet files
   - Contains information about map notes including their location and status

4. **notes_comments_data** - Comments on notes
   - Location: `../notes_comments_data/*.parquet`
   - Not partitioned, stored as flat parquet files
   - Join with notes_data using `note_id`

## Dataset Structure

### Changeset Data

The changeset dataset is located at `../changeset_data/year=*/month=*/*.parquet` and contains the following key columns:

### Base Columns
- `changeset_id` - ID of the changeset
- `edit_count` - Number of edits in the changeset
- `user_name` - OSM contributor username
- `year` - Year of the changeset
- `month` - Month of the changeset

### Enriched Columns (added by `scripts/changeset_raw_data_to_data.py`)
- `created_by` - Normalized editing software name
- `device_type` - Classification: `desktop_editor`, `mobile_editor`, `tool`, `other`
- `bot` - Boolean indicating if the changeset was made by a bot
- `mid_pos_x`, `mid_pos_y` - Discretized coordinates (0-360, 0-180)
- `imagery_used` - Array of imagery sources used
- `hashtags` - Array of hashtags from the changeset
- `source` - Array of data sources used
- `mobile_os` - Mobile OS detection (`Android`, `iOS`, or `NULL`)
- `streetcomplete_quest` - Normalized StreetComplete quest type
- `all_tags` - Array of all tag prefixes used
- `organised_team` - Organised team/corporation affiliation if applicable
- `for_profit` - Boolean indicating if the changeset was made by a for-profit organisation

### Changeset Comments Data

The changeset comments dataset is located at `../changeset_comments_data/*.parquet` and contains comments from OpenStreetMap changeset discussions:

**Columns:**
- `changeset_id` - ID of the changeset being commented on (join key with changeset_data)
- `date` - Timestamp when the comment was made (UTC)
- `user_name` - Username of the person who made the comment
- `text` - Content of the comment

### Notes Data

The notes dataset is located at `../notes_data/*.parquet` and contains information about map notes:

**Columns:**
- `note_id` - Unique identifier for the note
- `lat` - Latitude of the note location
- `lon` - Longitude of the note location
- `created_at` - Timestamp when the note was created (UTC)
- `closed_at` - Timestamp when the note was closed (NULL if still open)
- `mid_pos_x` - Discretized longitude coordinate (0-360)
- `mid_pos_y` - Discretized latitude coordinate (0-180)

### Notes Comments Data

The notes comments dataset is located at `../notes_comments_data/*.parquet` and contains comments on map notes:

**Columns:**
- `note_id` - ID of the note being commented on (join key with notes_data)
- `action` - Action type (e.g., "opened", "commented", "closed", "reopened")
- `timestamp` - Timestamp when the comment/action was made (UTC)
- `user_name` - Username of the person who made the comment/action
- `text` - Content of the comment

## Project Setup

### Standard Notebook Initialization

```python
import duckdb
import util

util.init()
```

## Core Patterns and Examples

### Pattern 1: Basic Time Series Analysis

**Example: Monthly Contributors/Edits/Changesets**

```python
df = duckdb.sql("""
WITH user_first_appearance AS (
    SELECT
        user_name,
        year,
        month,
        ROW_NUMBER() OVER (PARTITION BY user_name ORDER BY year, month) as rn
    FROM (
        SELECT DISTINCT user_name, year, month
        FROM '../changeset_data/year=*/month=*/*.parquet'
    )
),
first_appearances AS (
    SELECT user_name, year, month
    FROM user_first_appearance
    WHERE rn = 1
),
monthly_metrics AS (
    SELECT
        year,
        month,
        CONCAT(year, '-', LPAD(CAST(month as VARCHAR), 2, '0')) as months,
        COUNT(DISTINCT user_name) as Contributors,
        CAST(SUM(edit_count) as BIGINT) as Edits,
        CAST(COUNT(*) AS INTEGER) as Changesets
    FROM '../changeset_data/year=*/month=*/*.parquet'
    GROUP BY year, month
),
monthly_new_contributors AS (
    SELECT
        year,
        month,
        COUNT(DISTINCT user_name) as "New Contributors"
    FROM first_appearances
    GROUP BY year, month
),
combined_metrics AS (
    SELECT
        m.year,
        m.month,
        m.months,
        m.Contributors,
        COALESCE(n."New Contributors", 0) as "New Contributors",
        m.Edits,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [piebro/openstreetmap-statistics](https://github.com/piebro/openstreetmap-statistics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
