---
trigger: always_on
description: A Home Assistant custom integration for indexing and querying media files with EXIF metadata extraction, geocoding, and database caching.
---

# Media Index Integration

A Home Assistant custom integration for indexing and querying media files with EXIF metadata extraction, geocoding, and database caching.

## Project Overview

**Current Version**: v1.6.0
**Status**: Production ready
**Type**: Home Assistant Custom Integration (Python)

### Critical Rules

**🚨 DO NOT UPDATE VERSION NUMBERS IN THE MANIFEST.JSON OR SOURCE FILES DIRECTLY! YOU MUST ASK FOR PERMISSION FIRST! 🚨**

Version numbers are only updated when creating official releases. Changes accumulate in the current version until release time.

## Project Structure

- `custom_components/media_index/` - Main integration code
  - `__init__.py` - Integration setup, service registration, config flow
  - `cache_manager.py` - SQLite database operations, queries
  - `sensor.py` - Sensor entity for scan status
  - `scanner.py` - File scanning and EXIF extraction
  - `watcher.py` - File system watcher for automatic updates
  - `geocode.py` - Reverse geocoding for GPS coordinates
  - `const.py` - Constants and service names
  - `manifest.json` - Integration metadata and dependencies
- `scripts/` - Deployment and testing scripts
  - `deploy-media-index.ps1` - **CRITICAL**: Use this for all deployments
- `docs/` - User documentation
- `dev-docs/` - Architecture and implementation notes

## Architecture

### Core Components

1. **CacheManager** (`cache_manager.py`)
   - SQLite database operations
   - Two tables: `media_files` (file metadata) and `exif_data` (EXIF details)
   - Compound cursor pagination for stable ordered queries
   - Foreign keys enabled with `PRAGMA foreign_keys = ON`

2. **Scanner** (`scanner.py`)
   - EXIF extraction from images (piexif)
   - Video metadata extraction (pymediainfo - requires libmediainfo system library)
   - Batch scanning with progress reporting
   - Async file operations using executor threads

3. **MediaWatcher** (`watcher.py`)
   - File system monitoring using watchdog
   - Event batching and throttling (2s delay, 50 file batches)
   - Thread-safe queue operations with `call_soon_threadsafe`
   - Processes deletions first, then new files, then modifications

4. **Geocoding** (`geocode.py`)
   - Reverse geocoding using Nominatim
   - SQLite cache for API results
   - Batch statistics tracking (100 lookup batches)
   - Language support (native or HA instance language)

### Database Schema

**NEVER INVENT COLUMN NAMES** - Always check the CREATE TABLE statements in cache_manager.py first!

**media_files table:**
- `id` (INTEGER PRIMARY KEY)
- `path` (TEXT UNIQUE)
- `filename` (TEXT)
- `folder` (TEXT)
- `file_size` (INTEGER)
- `width`, `height` (INTEGER)
- `duration` (REAL) - video duration in seconds
- `file_type` (TEXT) - image/video file type
- `created_time` (INTEGER) - Unix timestamp when file was created
- `modified_time` (INTEGER) - Unix timestamp when file was last modified
- `orientation` (INTEGER) - EXIF orientation value or normalized rotation
- `last_scanned` (INTEGER) - Unix timestamp when this file was last scanned
- `rating` (INTEGER) - 0-5 stars
- `rated_at` (INTEGER) - Unix timestamp when rating was last updated

**exif_data table:**
- `id` (INTEGER PRIMARY KEY)
- `file_id` (INTEGER FOREIGN KEY → media_files.id ON DELETE CASCADE)
- `date_taken` (TEXT) - ISO 8601 timestamp when the media was captured
- `is_favorited` (INTEGER) - 0/1 boolean (default 0)
- `camera_make`, `camera_model` (TEXT)
- `iso`, `aperture`, `focal_length`, `shutter_speed` (TEXT/REAL)
- `latitude`, `longitude`, `altitude` (REAL)
- `location_name`, `location_city`, `location_state`, `location_country` (TEXT) - geocoded location
- `focal_length_35mm`, `exposure_compensation`, `metering_mode`, `white_balance`, `flash` (TEXT)
- `rating` (INTEGER) - 0-5 stars

## Development Guidelines

### CRITICAL: Code Verification Before Writing

**Database Queries:**
- **NEVER invent column names** - grep for CREATE TABLE statements
- **NEVER assume method exists** - search for actual method definition
- **ALWAYS check existing queries** - find similar patterns before writing new code

**Example Mistake:**
```python
# ❌ WRONG - invented column names
SELECT m.extension, m.size, m.date_modified FROM media_files m

# ✅ CORRECT - actual column names
SELECT m.filename, m.file_size, m.modified_time FROM media_files m
```

**Service Names:**
- Check `const.py` for all SERVICE_* constants
- Check `services.yaml` for actual service definitions
- Never invent service names or parameters

**Before Writing ANY Code:**
1. Read the actual implementation you're modifying
2. Search for similar patterns (grep_search)
3. Verify schema/structure by reading relevant files
4. Check existing tests or usage examples
5. Only then write code matching actual patterns

### Logging Best Practices

**Per-Item vs Batch Logging:**
- Use `DEBUG` level for per-item operations (file processing, individual queries)
- Use `INFO` level for batch summaries (e.g., "Processing 50 new files")
- Use `WARNING` for recoverable errors
- Use `ERROR` for failures requiring attention

**Example:**
```python
# ❌ WRONG - logs INFO for every file (causes "logging too frequently")
for file_path in files:
    _LOGGER.info("Processing %s", file_path)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [markaggar/ha-media-index](https://github.com/markaggar/ha-media-index) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
