---
trigger: always_on
description: A local web application for managing, tagging, sorting, and censoring Stable Diffusion generated images. Runs as a FastAPI backend serving a vanilla HTML/JS/CSS frontend on `127.0.0.1:8487` by default (configurable via `SD_IMAGE_SORTER_PORT`).
---

# SD Image Sorter

## Project Overview

A local web application for managing, tagging, sorting, and censoring Stable Diffusion generated images. Runs as a FastAPI backend serving a vanilla HTML/JS/CSS frontend on `127.0.0.1:8487` by default (configurable via `SD_IMAGE_SORTER_PORT`).

## Target Platform & Supported Resolutions — READ FIRST

**This is desktop/laptop computer software. ONLY support laptop and desktop screen resolutions. Do NOT spend any effort on mobile or tablet.**

- ✅ **In scope:** laptop + desktop viewports, roughly **1280px wide and up** — e.g. 1280×720/800 (small laptop), 1366×768 (most common laptop), 1440×900, 1536×864, 1920×1080, 2560×1440, 3440×1440 (ultrawide), 3840×2160 (4K). Test, audit, and optimize here.
- ❌ **OUT OF SCOPE — do not test, audit, optimize, screenshot, or "fix" anything here:** phones and tablets — any width below ~1280px (320 / 375 / 390 / 414 / 768 / 1024). Do not add new mobile/tablet responsive work. Do not run responsive sweeps at these widths.
- The global rule `~/.Codex/rules/web/testing.md` lists breakpoints `320 / 375 / 768 / 1024 / 1440 / 1920` and "screenshot key breakpoints 320/768/1024/1440". **For THIS project that is overridden** — ignore everything below ~1280px; use desktop/laptop widths only (1366, 1440, 1920, 2560, 3840).
- Existing mobile/tablet responsive CSS already in the tree (e.g. censor stacking ≤960px) is harmless — **leave it as-is; do not invest more in it and do not rip it out** (removing it is itself wasted mobile effort and risks regressions). Width-agnostic fixes that also improve desktop (e.g. empty-state cleanups) are fine.

> Owner directive (2026-06-05, stated emphatically and repeatedly): "We are computer software. Just take care of laptop and desktop resolutions." Auditing phone/tablet widths was explicitly called out as wasted effort. Honor this for ALL UI/UX, layout, testing, and review work.

## Architecture

```
Browser (127.0.0.1:8487 default)
  |
  | HTTP REST API
  v
FastAPI (backend/main.py)
  - application assembly, service initialization, router mounting
  |
  +-- app_static.py       → GET /, /static/*, JS/CSS cache busting
  +-- app_security.py     → CORS, localhost-only guard, rate limit, security headers
  +-- app_diagnostics.py  → support log diagnostics and file-manager open flow
  |
  +-- /api/*              → REST endpoints via routers
  |
  +-- routers/images.py      → image retrieval, thumbnails
  +-- routers/tags.py       → AI tagging (WD14 ONNX), tag CRUD
  +-- routers/sorting.py    → scan folders, move/batch-move, WASD manual sort
  +-- routers/censor.py     → YOLOv8 detection + Pillow censoring
  +-- routers/prompts.py    → prompt generation endpoints
  +-- routers/similarity.py → CLIP embedding similarity search
  +-- routers/artists.py    → artist identification (experimental)
  |
  +-- database.py         → SQLite (raw SQL, no ORM)
  +-- metadata_parser.py  → SD metadata extraction (ComfyUI/NAI/WebUI/Forge)
  +-- image_manager.py    → file operations (scan, move, copy)
  +-- tagger.py           → WD14 tagger via ONNX Runtime
  +-- censor.py           → YOLOv8 ONNX + Pillow
  +-- services/           → business logic and feature orchestration
```

## Tech Stack

- **Backend**: Python 3.12+, FastAPI, Uvicorn, SQLite, Pillow, ONNX Runtime
- **Frontend**: Vanilla HTML5 / CSS3 / JavaScript (no framework, no build step)
- **AI Models**: WD14 Tagger (ONNX, from HuggingFace), YOLOv8 (segmentation, .pt/.onnx)
- **UI Style**: Glassmorphism (backdrop-filter, translucency, blur)

## Quick Start

```bash
# Windows
run.bat

# Linux/Mac
./run.sh
```

Both scripts auto-create a Python venv in `backend/venv/` and install dependencies on first run.

## Project Structure

```
sd-image-sorter/
├── backend/
│   ├── main.py               # FastAPI app entry point
│   ├── app_security.py       # Security middleware wiring
│   ├── app_static.py         # Static frontend serving and cache busting
│   ├── app_diagnostics.py    # Support diagnostics and log opening
│   ├── database.py           # SQLite layer (raw SQL)
│   ├── metadata_parser.py    # SD image metadata extraction
│   ├── image_manager.py      # File operations (scan, move, copy)
│   ├── tagger.py             # WD14 AI tagger (ONNX Runtime)
│   ├── censor.py             # YOLOv8 detection + censoring
│   ├── requirements.txt      # Python dependencies
│   ├── routers/
│   │   ├── images.py         # GET /api/images, /api/image-file, /api/image-thumbnail
│   │   ├── tags.py           # Tag CRUD, tagging pipeline, library endpoints
│   │   ├── sorting.py        # Scan, move, batch-move, manual sort (WASD)
│   │   ├── censor.py         # YOLO detect, preview, save endpoints
│   │   ├── prompts.py        # Prompt generation endpoints
│   │   ├── similarity.py     # CLIP embedding similarity search
│   │   ├── artists.py        # Artist identification (experimental)
│   ├── services/
│   │   ├── image_service.py          # Image workflows
│   │   ├── image_metadata_writer.py  # Reader metadata save helpers
│   │   ├── sorting_service.py        # Sorting workflow orchestration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [peter119lee/sd-image-sorter](https://github.com/peter119lee/sd-image-sorter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
