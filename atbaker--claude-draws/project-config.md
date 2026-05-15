---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Claude Draws** is an automated art project where Claude for Chrome creates crowdsourced illustrations using Kid Pix, sourced from user submissions at claudedraws.xyz. The complete workflow:

1. **Web form** (claudedraws.xyz/submit) accepts art requests and stores them in Cloudflare D1
2. **Browser automation** (Playwright + CDP) submits prompts to Claude for Chrome extension
3. **OBS recording** captures the entire artwork creation process via WebSocket control
4. **Claude draws** in a modified Kid Pix JavaScript app (served from local directory, not included in this repo)
5. **Temporal workflows** orchestrate the post-processing pipeline
6. **BAML extraction** parses Claude's title and artist statement from the final response
7. **Cloudflare R2** stores artwork images and creation videos
8. **Cloudflare D1** stores artwork metadata (including video URLs) in SQLite database
9. **SvelteKit gallery** (`gallery/`) displays all artworks with videos at claudedraws.xyz with SSR-on-demand
10. **Cloudflare Workers** hosts the gallery site

### Repository Structure (Monorepo)

This repository uses a monorepo structure to organize different components:

```
claude-draws/
├── backend/              # Python backend (CLI, Temporal workflows, BAML)
│   ├── claudedraw/      # CLI tool for browser automation
│   ├── workflows/       # Temporal workflow definitions
│   ├── worker/          # Temporal worker process
│   ├── baml_src/        # BAML definitions for metadata extraction
│   ├── pyproject.toml   # Python dependencies
│   └── Dockerfile.worker # Container for worker
├── gallery/             # SvelteKit frontend (SSR-on-demand with D1 API)
├── .chrome-data/        # Chrome profile for automation (gitignored)
├── downloads/           # Temporary artwork storage (gitignored)
├── docs/               # Documentation
└── docker-compose.yml  # Orchestrates all services
```

### Key Components

1. **Python CLI tool** (`backend/claudedraw/`) - Browser automation to submit prompts to Claude for Chrome
2. **Temporal workflows** (`backend/workflows/`) - Orchestrates artwork processing, metadata extraction, R2 image upload, and D1 metadata insertion
3. **Temporal worker** (`backend/worker/`) - Runs the Temporal worker process that executes workflow activities
4. **SvelteKit gallery** (`gallery/`) - SSR-on-demand site with D1 API backend, deployed to Cloudflare Workers
5. **BAML integration** (`backend/baml_src/`) - AI-powered extraction of artwork titles and artist statements

## Key Architecture Details

### Temporal Workflow Architecture

**Primary Workflow**: `backend/workflows/create_artwork.py` - `CreateArtworkWorkflow`

The workflow handles the **complete end-to-end process**:

1. **Starts OBS recording** - Begins recording the artwork creation process for the gallery
2. **Browser automation** - Finds pending submission from D1, submits to Claude, waits for completion
3. **Stops OBS recording** - Ends recording and retrieves video file path
4. **Extracts metadata** using BAML - Parses Claude's HTML response to extract artwork title and artist statement
5. **Uploads image to R2** - Stores PNG file in Cloudflare R2 bucket with public access
6. **Uploads video to R2** - Stores MKV/MP4 recording in R2 bucket with public access
7. **Inserts metadata into D1** - Stores artwork metadata (including video URL) in D1 database (artwork appears immediately in gallery)
8. **Sends email notification** - Notifies requester when artwork is ready (if email provided)
9. **Schedules next workflow** (continuous mode only) - Enables livestream operation

**Key activities** in `backend/workflows/activities.py`:
- `start_obs_recording()` - Starts OBS recording to the recordings directory
- `browser_session_activity()` - Long-running activity that automates the browser (find submission → submit → wait → download)
- `stop_obs_recording()` - Stops OBS recording and returns the video file path
- `compress_video()` - Compresses video using PyAV (H.264, ~70-75% size reduction)
- `extract_artwork_metadata()` - Uses BAML to parse Claude's response HTML
- `upload_image_to_r2()` - Uploads PNG to R2 with public access
- `upload_video_to_r2()` - Uploads compressed video to R2 with public access
- `insert_artwork_to_d1()` - Inserts artwork metadata into D1 database
- `send_email_notification()` - Sends completion email to requester (if email provided)

**Browser Automation Details** (implemented in `browser_session_activity()`):

1. **Environment variable must be set BEFORE importing Playwright**: `os.environ['PW_CHROMIUM_ATTACH_TO_OTHER'] = '1'`
   - This is critical - it enables Playwright's Node.js server to attach to Chrome extension side panels (which are targets of type "other")

2. **Hybrid automation approach**:
   - Playwright connects to Chrome via CDP (Chrome DevTools Protocol)
   - OS-level keyboard automation via `pyautogui` is required to trigger browser extension shortcuts (Command+E to open Claude side panel)
   - Playwright's `page.keyboard.press()` does NOT work for extension shortcuts - only affects page content

3. **Finding the side panel**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [atbaker/claude-draws](https://github.com/atbaker/claude-draws) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
