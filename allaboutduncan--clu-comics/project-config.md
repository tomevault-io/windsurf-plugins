---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Comic Library Utilities (CLU) is a Flask-based web application for managing comic book collections. It provides bulk operations for CBZ/CBR files, metadata editing, file renaming, format conversion, and folder monitoring. Designed to run in Docker, it integrates with comic databases (GCD, ComicVine, Metron) for metadata enrichment.

## Development Commands

```bash
# Run locally (development)
python app.py

# Run with Docker
docker build -t comic-utils .
docker run -p 5577:5577 -v /path/to/comics:/data -v /path/to/downloads:/downloads comic-utils

# Verify Python syntax
python -m py_compile <filename.py>

# Production server (used in Docker)
gunicorn -w 1 --threads 8 -b 0.0.0.0:5577 --timeout 120 app:app
```

## Architecture

### Core Application Flow
- **`api.py`**: Creates the Flask app instance and handles download queue/remote downloads
- **`app.py`**: Main application - imports Flask app from `api.py`, registers blueprints, defines all routes and API endpoints
- **`monitor.py`**: Standalone file watcher for folder monitoring (runs when `MONITOR=yes`)

### Core Modules (`core/`)
| Module | Purpose |
|--------|---------|
| `core/config.py` | ConfigParser-based settings from `/config/config.ini` |
| `core/database.py` | SQLite database (`comic_utils.db`) for caching, file index, reading history |
| `core/comicinfo.py` | ComicInfo.xml parsing and generation |
| `core/app_logging.py` | Centralized logging — `app_logger` and `monitor_logger`, log files in `CONFIG_DIR/logs` |
| `core/app_state.py` | Global state — APScheduler instance, wanted-issues refresh state, data-dir stats cache |
| `core/file_watcher.py` | DebouncedFileHandler for `/data` monitoring — detects changes, queues metadata scanning |
| `core/metadata_scanner.py` | Background worker scanning ComicInfo.xml — priority queue, updates file_index with metadata |
| `core/memory_utils.py` | Memory monitoring — tracks usage, triggers cleanup at thresholds, `memory_context()` manager |
| `core/version.py` | Single `__version__` string |

### Other Root Modules
| Module | Purpose |
|--------|---------|
| `rename.py` | Comic file renaming with regex patterns for volume/issue extraction |
| `edit.py` | CBZ editing - image manipulation, file reordering, cropping |
| `convert.py` | CBR to CBZ conversion using `unar` |
| `wrapped.py` | Yearly reading stats image generation (Spotify Wrapped style) |
| `helpers.py` | Utility functions — `is_hidden()`, `safe_image_open()`, `create_thumbnail_streaming()`, ZIP/RAR extraction |
| `recommendations.py` | AI-powered recommendations via OpenAI/Anthropic APIs |

### Models
| Module | Purpose |
|--------|---------|
| `models/metron.py` | Metron API via Mokkari — search, metadata fetch, rate-limit retry, scrobble |
| `models/comicvine.py` | ComicVine API via Simyan — volume/issue search, metadata mapping |
| `models/gcd.py` | Grand Comics Database — MySQL queries, fuzzy title matching |
| `models/komga.py` | Komga media server REST client — reading history, in-progress books |
| `models/getcomics.py` | GetComics.org scraper — cloudscraper-based search and download |
| `models/mega.py` | MEGA download support — URL parsing, AES-256 decryption |
| `models/stats.py` | Library statistics — file counts, disk usage, read stats (cached) |
| `models/timeline.py` | Reading timeline — groups history by date, filters by year/month |
| `models/cbl.py` | CBL (Comic Book List) XML parser — matches entries to collection files |
| `models/issue.py` | Data classes — `IssueObj` and `SeriesObj` for unified data representation |
| `models/update_xml.py` | Batch ComicInfo.xml field updater across CBZ files |
| `models/providers/` | Unified provider system — `BaseProvider` ABC, registry, adapters for Metron/ComicVine/GCD/AniList/MangaDex/Bedetheque |

### CBZ Operations
| Module | Purpose |
|--------|---------|
| `cbz_ops/add.py` | Insert blank images into CBZ files |
| `cbz_ops/delete.py` | Delete CBZ files from filesystem |
| `cbz_ops/convert.py` | CBR→CBZ conversion using `unar` |
| `cbz_ops/single_file.py` | Single RAR→CBZ conversion with progress reporting |
| `cbz_ops/edit.py` | CBZ editing — crop, reorder, extract covers |
| `cbz_ops/crop.py` | Cover image cropping — left/center/right/freeform with blur |
| `cbz_ops/remove.py` | Remove specific images from CBZ files |
| `cbz_ops/enhance_single.py` | Single image enhancement — contrast, brightness, blur |
| `cbz_ops/enhance_dir.py` | Batch directory image enhancement |
| `cbz_ops/rebuild.py` | Rebuild CBZ structure — normalize filenames, reorder images |
| `cbz_ops/pdf.py` | PDF→CBZ conversion via pdf2image |
| `cbz_ops/rename.py` | Comic file renaming with regex pattern matching |

### Routes
| Module | Purpose |
|--------|---------|
| `routes/downloads.py` | GetComics search/download, auto-download schedules, weekly packs |
| `routes/files.py` | File ops — rename, delete, move, crop, combine CBZ, upload, cleanup |
| `routes/collection.py` | File browsing — directory listing, search, thumbnails, metadata browse |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [allaboutduncan/clu-comics](https://github.com/allaboutduncan/clu-comics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
