---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Surveillance video analysis system ("Споглядайко") that monitors a folder for new `.mp4` files, detects motion and objects, generates AI descriptions via Google Gemini, and sends results to Telegram. Designed for a residential gate/entrance camera with gate crossing detection, person re-identification, and optional Tesla integration.

Companion Android dashboard app: [analyze-video-dashboard](https://github.com/alivespirit/analyze-video-dashboard) (Jetpack Compose, consumes the JSON API from `tools/log_dashboard/`).

## Running the Application

```bash
# Setup
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

# Run (requires .env with GEMINI_API_KEY, TELEGRAM_TOKEN, TELEGRAM_CHAT_ID, VIDEO_FOLDER)
python main.py
```

The app auto-restarts when any `.py` file is modified (via watchdog + `os.execv`).

## Testing a Single Video

```bash
python tools/run_detect_motion.py /path/to/video.mp4 [output_dir] [--log-level DEBUG]
```

There is no automated test suite. Validation is done via `tools/validate_log_replay.py` for batch re-analysis.

## Architecture

**Pipeline**: File monitoring → Motion detection (local or remote worker) → AI analysis → Telegram notification

**Dual-executor pattern** (critical design choice):
- `motion_executor`: ThreadPoolExecutor(max_workers=1) — CPU-bound local motion detection runs serially
- `io_executor`: ThreadPoolExecutor(max_workers=4) — I/O-bound Gemini/Telegram calls run in parallel
- Remote worker dispatch is fully async (no executor slot consumed while waiting on HTTP)
- Both executors are driven by a single asyncio event loop in `main.py`

### Core Modules

| Module | Role |
|--------|------|
| `main.py` | Entry point, asyncio orchestration, watchdog file monitoring, Telegram bot setup, processing ledger, auto-restart, retention cleanup, Tesla SoC scheduler |
| `detect_motion.py` | Core vision: background subtraction, YOLO tracking (OpenVINO), gate crossing detection, ReID triggering, highlight clip generation, car speed estimation |
| `analyze_video.py` | Gemini API calls with dynamic model selection (time-based Pro vs Flash), fallback chains, prompt loading from `config/prompt.txt` |
| `telegram_notification.py` | Message delivery with retry logic, grouped notifications, inline callbacks for full video, media validation. Respects `KEEP_HIGHLIGHTS_CLIPS` for clip cleanup. |
| `person_id.py` | Intel OpenVINO ReID model, gallery embedding with disk caching (keyed by gallery path + model path), negative gallery support, cosine similarity matching |
| `path_utils.py` | Timestamp extraction from video filenames |
| `worker/client.py` | Master-side worker dispatch: async HTTP, health check with caching, local fallback (lazy YOLO load), log replay with `[W]` tagging, Wake-on-LAN |
| `worker/server.py` | FastAPI worker server: path translation (Windows↔Linux), video copy, detect_motion, result copy back to CIFS, gallery cache pre-warming |
| `tools/log_dashboard/app.py` | FastAPI dashboard: HTML log viewer + JSON API for Android app (per-video summaries, stats, monitoring, events, image/clip serving, ReID gallery management) |

### Data Flow

1. `main.py` watchdog detects new `.mp4` → checks worker availability
2. If worker available: async HTTP dispatch to `worker/server.py`; otherwise queues to `motion_executor` locally
3. `detect_motion.py` returns result dict with status, clip path, person/car counts, crossing info, ReID results
4. `analyze_video.py` decides whether to call Gemini based on motion status and time of day
5. `telegram_notification.py` sends animation/photo/text with retry and grouping logic

### Directory Structure

```
analyze-video/
├── main.py, detect_motion.py, analyze_video.py   # core pipeline
├── telegram_notification.py, person_id.py        # core pipeline
├── path_utils.py                                 # utility
├── config/                                       # runtime configuration files
│   ├── roi.json, roi-1080p.json, roi-4k.json     # ROI polygons
│   ├── tracker.yaml                              # ByteTrack/BoTSORT config
│   ├── gemini_models.env                         # Gemini model names (hot-reload)
│   └── prompt.txt                               # Gemini prompt (Ukrainian)
├── worker/                                       # remote worker package
│   ├── server.py                                 # FastAPI worker server
│   ├── client.py                                 # master-side dispatch client + WOL
│   ├── worker.service                            # systemd unit
│   └── requirements.txt                          # worker-only dependencies
├── tools/                                        # dev/ops utilities
│   └── log_dashboard/                            # HTML dashboard + JSON API
│       ├── app.py                                # FastAPI app (HTML routes + /api/* JSON routes)
│       ├── templates/                            # Jinja2 HTML templates
│       └── static/                               # CSS
├── models/                                       # YOLO + ReID model files

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alivespirit) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
