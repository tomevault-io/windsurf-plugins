---
trigger: always_on
description: validates that synthetic sunrise produces expected monotonically increasing brightness.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Video stream capture and analysis pipeline of an RTSP stream. Captures streams via OpenCV, stores timelapsed frames in HDF5, and performs image analysis to detect brightness changes for tracking sourdough starter activity across feedings. Supports **cultures** — named groups of sessions for cross-feeding analysis.

## Project Structure

```
src/breadcam/
├── config.py           # dataclass-based settings, loads .env
├── capture.py          # RTSP single-frame grab via OpenCV
├── processing.py       # single-frame transformations (grayscale, blur, normalize, diff)
├── analysis.py         # batch frame analysis against anchor frames
├── storage.py          # HDF5 read/write (schema v2.0)
├── session.py          # session lifecycle orchestration
├── scheduler.py        # periodic capture loop
├── visualization.py    # GIF animation generation
├── synthetic_frames.py # shared synthetic sunrise utilities (testing/synthetic server)
├── commands.py         # programmatic API for all CLI operations
├── cli.py              # CLI entry point (breadcam command)
├── camera_store.py     # thread-safe camera registry (data/cameras.json)
├── culture_store.py    # thread-safe culture registry (data/cultures.json)
├── culture_analysis.py # cross-session analysis: SessionSummary + CultureAnalysis
├── utils.py            # session ID generation
├── time_parsing.py     # human-readable duration/interval parsing for CLI
└── api/
    ├── main.py         # FastAPI app factory, lifespan, CORS, static mounts
    ├── config.py       # APISettings dataclass (host, port, log_level, etc.)
    ├── models.py       # Pydantic request/response schemas, NaN-handling validators
    ├── dependencies.py # FastAPI DI with LRU caching (settings, camera store, culture store, task manager)
    ├── tasks.py        # background task lifecycle (TaskManager, CaptureTask, TaskStatus)
    ├── file_manager.py # temp file storage and auto-cleanup (gifs, images, sessions)
    ├── static/         # control.html/js/css, dashboard.html/js/css
    └── routers/
        ├── health.py   # GET /api/v1/health, GET /api/v1/config
        ├── grab.py     # POST /api/v1/camera/grab
        ├── cameras.py  # CRUD /cameras + POST /cameras/{id}/activate
        ├── captures.py # POST/GET/POST /captures (start, status, cancel); culture_id auto-assign
        ├── sessions.py # GET/POST /sessions (list filterable by culture, info w/ culture_id, analyze, file, analysis, frames)
        ├── cultures.py # CRUD /cultures + session assign/unassign + GET /cultures/{id}/analysis
        ├── control.py  # GET /control (camera management + culture management UI)
        ├── dashboard.py# GET /dashboard (analytics UI with culture selector)
        └── files.py    # GET /files/{category}/{filename}
```

## Commands

### Install dependencies
```bash
uv sync --all-extras
```

### Grab a single frame
**Requires synthetic RTSP server running** (see synthetic RTSP Server section).
```bash
export BREADCAM_DATA_ENV=dev
uv run breadcam grab -o frame.png
```

### Run a capture session
**Requires synthetic RTSP server running** (see synthetic RTSP Server section).
```bash
export BREADCAM_DATA_ENV=dev
uv run breadcam capture --duration 1.0 --interval 15
```

### Run a capture session with raw frame storage
**Requires synthetic RTSP server running** (see synthetic RTSP Server section).
```bash
export BREADCAM_DATA_ENV=dev
uv run breadcam capture --duration 1.0 --interval 15 --store-raw
```

**Note:** Enabling `--store-raw` stores both preprocessed frames and full-resolution raw BGR frames in the HDF5 session file. This increases file size by approximately 50-100% but enables:
- Debugging of the preprocessing pipeline (compare raw vs processed)
- Alternative analysis workflows requiring original color information
- ML/CV experiments needing full-resolution color data
- Post-capture reprocessing with different parameters

Use `--store-raw` only when raw frames are needed for specific analysis or debugging purposes.

### View session info
```bash
uv run breadcam info data/sessions/<session>.h5
```

### Generate visualization
```bash
uv run breadcam analyze data/sessions/<session>.h5 -o output.gif
```

### Generate visualization with custom anchor frame
```bash
uv run breadcam analyze data/sessions/<session>.h5 -o output.gif --anchor-frame 2
```

### Run tests
```bash
# Run all tests (including integration tests)
uv run pytest

# Run only integration tests
uv run pytest -m integration

# Skip integration tests (faster for development)
uv run pytest -m "not integration"

# Skip slow tests
uv run pytest -m "not slow"

# Run specific test file
uv run pytest tests/test_visualization_integration.py -v
```

### Run linting/formatting
```bash
uv run ruff check src/ tests/
uv run ruff format src/ tests/
```

### Run notebooks
```bash
uv sync --all-extras
uv run jupyter notebook
```

## Data Environment Management

The project supports environment-based data directory separation via the `BREADCAM_DATA_ENV` environment variable. This allows you to keep production captures separate from development/test data.

### Environments


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ddoyled/breadcam](https://github.com/ddoyled/breadcam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->
