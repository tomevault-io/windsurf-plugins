---
trigger: always_on
description: This file provides guidance for AI agents working with code in this repository.
---

# AGENTS.md

This file provides guidance for AI agents working with code in this repository.

## Project Overview

Sonarr Metadata Rewrite - A compatibility layer that monitors
Sonarr-generated .nfo files and overwrites them with TMDB translations in
desired languages, and rewrites poster/clearlogo images to language-specific
variants when available. This addresses [Sonarr Issue #269](
https://github.com/Sonarr/Sonarr/issues/269) (multilingual metadata) and
[Sonarr Issue #6663](https://github.com/Sonarr/Sonarr/issues/6663)
(localized images).

The project includes a metadata translation service and an image rewrite
pipeline with real-time file monitoring, TMDB API integration, intelligent
caching, comprehensive error handling, and reprocessing avoidance.

## Essential Development Commands

### Setup and Environment

```bash
# Initial setup (installs dependencies and pre-commit hooks)
./scripts/setup-dev.sh

# Install project in development mode after dependencies change
uv sync --group dev

# Create .env file with required API key for development and testing
echo "TMDB_API_KEY=your_api_key_here" > .env
```

### Code Quality and Testing

```bash
# Run all linting checks and fixes
./scripts/lint.sh

# Run checks only (no auto-fixes)
./scripts/lint.sh --check

# Run unit tests with coverage
./scripts/run-unit-tests.sh

# Run integration tests
./scripts/run-integration-tests.sh

# Combine coverage from unit and integration tests
./scripts/combine-coverage.sh
```

### Development Usage

```bash
# Run in development mode (requires .env file with TMDB_API_KEY)
uv run sonarr-metadata-rewrite

# Check version
uv run sonarr-metadata-rewrite --version

# Run with custom settings for testing
REWRITE_ROOT_DIR=/tmp/test/media \
PREFERRED_LANGUAGES='zh-CN,ja-JP' \
CACHE_DURATION_HOURS=1 \
uv run sonarr-metadata-rewrite
```

## Technical Architecture

### Current Implementation

Metadata translation and image rewrite service with Click framework providing:

- Entry point: `sonarr_metadata_rewrite.main:cli` command that runs a
  persistent service (CLI: `sonarr-metadata-rewrite`)
- Real-time file monitoring with watchdog for immediate processing (.nfo and
  rewritable images)
- Periodic directory scanning for batch processing of existing files
- TMDB API integration with intelligent caching and exponential backoff retry
  for rate limit handling
- Reprocessing avoidance to prevent unnecessary file updates and API calls
- Pydantic-based configuration with comprehensive settings validation
- Atomic file operations with optional backup functionality
- Project structure follows Python package standards with src/ layout and
  _version.py

### Core Components

1. **CLI Interface** (`main.py`)
Click-based command that runs a persistent service. Performs comprehensive
configuration validation on startup, installs signal handlers for graceful
shutdown, starts the service synchronously, and uses proper exit codes on
failure.

1. **RewriteService** (`rewrite_service.py`)
Orchestrator coordinating metadata and image processing components. Manages
file monitoring, scanning, and processing lifecycle, and handles
startup/shutdown and resource cleanup.

1. **MetadataProcessor** (`metadata_processor.py`)
Complete .nfo file processing workflow: extract TMDB IDs from XML, select
translation based on language preferences, and write atomically with optional
backup creation.

1. **Translator** (`translator.py`)
TMDB API client using httpx with diskcache-backed caching and exponential
backoff for HTTP 429. Supports series and episode translations. For images,
fetches `/tv/{id}/images` or `/tv/{id}/season/{s}/images`, filters locally for
preferred language-country (e.g., `en-US`) and chooses the first exact match;
does not pass `include_image_language` due to TMDB API issues.

1. **FileMonitor** (`file_monitor.py`)
Real-time file system monitoring using watchdog. Processes .nfo and image files
on close/move events. Recursive directory watching.

1. **FileScanner** (`file_scanner.py`)
Periodic directory scanning for batch processing. Scans for both `.nfo` files
and rewritable images. Configurable scan intervals, thread-based background
scanning with graceful shutdown.

1. **Configuration** (`config.py`)
Pydantic Settings-based configuration with custom env source parsing
`PREFERRED_LANGUAGES` as a comma-separated string (not JSON) into `list[str]`.
Includes `ENABLE_IMAGE_REWRITE` flag (default: true). Loads from .env and
performs comprehensive validation.

1. **ImageProcessor** (`image_processor.py`)
Complete image rewrite workflow: identify rewritable image files (poster,
clearlogo, season posters), extract TMDB IDs, fetch image candidates via
Translator, select best match based on language preferences, download and
rewrite with embedded marker, and atomic writes with optional backup.

1. **ImageUtils** (`image_utils.py`)
Image metadata helpers for embedding and reading markers in PNG (tEXt chunk) and
JPEG (EXIF UserComment) formats. Markers contain TMDB file_path and language info
to enable reprocessing avoidance.

1. **RollbackService** (`rollback_service.py`)
Backup restoration service that reverts .nfo and image files to their original
state. Removes conflicting extension variants during image rollback.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kfstorm/sonarr-metadata-rewrite](https://github.com/kfstorm/sonarr-metadata-rewrite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
