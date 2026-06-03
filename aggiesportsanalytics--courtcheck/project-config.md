---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Always-Active Skills

These skills must be followed at all times, without exception:

- **`verification-before-completion`** — Never claim work is done, fixed, or passing without running verification commands first. Evidence before assertions, always.
- **`test-driven-development`** — Write the test before the implementation for any new feature or bug fix.
- **`systematic-debugging`** — Always find the root cause before proposing a fix. No patches without investigation first.

## Project Overview

CourtCheck is a tennis video analysis application that uses computer vision and machine learning to analyze tennis match recordings. The system detects balls, court boundaries, bounces, and strokes, then overlays tracking data and a minimap on the processed video.

## Architecture

This is a full-stack monorepo with two main components:

### Backend (`/backend`)
- **Language**: Python 3.10
- **Deployment**: Modal serverless platform with GPU (A10G)
- **Entry Point**: `backend/app.py` - Modal function that downloads video from Supabase, runs pipeline, uploads results
- **Core Pipeline**: `backend/pipeline/run.py` - Two-pass video processing (tracking, then drawing)
- **Dependencies**: PyTorch, Ultralytics (YOLO), OpenCV, CatBoost, FastAPI

**Key Modules**:
- `backend/models/` - ML model wrappers:
  - `ball_tracker.py` - YOLO-based ball detection (TrackNet)
  - `court_line_detector.py` - Court keypoint detection network
  - `bounce_detector.py` - CatBoost-based bounce detection
  - `stroke_detector.py` - Stroke/action recognition
  - `player_tracker.py` - YOLOv8x player detection and tracking with player filtering
- `backend/vision/` - Computer vision utilities:
  - `homography.py` - Court perspective transformation
  - `court_reference.py` - Reference court diagram generation
  - `drawing.py` - Overlay rendering (ball traces, court lines, minimap)
  - `heatmaps.py` - Heatmap generation (bounce + player position heatmaps using histogram2d + gaussian blur)
  - `postprocess.py` - Post-processing utilities
- `backend/pipeline/` - Pipeline orchestration:
  - `run.py` - Main pipeline execution
  - `storage.py` - Supabase integration, video upload/download, ffmpeg streamable conversion
  - `config.py` - Configuration

**Model Weights**: Stored in `backend/weights/` (not checked into git). Required files:
- `tracknet_weights.pt` - Ball detection YOLO model
- `keypoints_model.pth` - Court keypoint detection model
- `bounce_detection_weights.cbm` - CatBoost bounce model
- `stroke_classifier_weights.pth` - Stroke recognition model
- `yolov8x.pt` - YOLOv8x player detection model (auto-downloads on first run)

### Frontend (`/frontend/web`)
- **Framework**: Next.js 16 (React 19) with TypeScript
- **UI**: shadcn/ui components with Tailwind CSS
- **Routing**: App Router (Next.js 13+ pattern)
- **State**: React Hook Form with Zod validation
- **Data**: Supabase client for database and storage

**Page Routes**:
- `/` - Dashboard (main page)
- `/upload` - Video upload interface
- `/recordings` - View processed recordings list
- `/recordings/[id]` - Watch processed video with heatmaps
- `/match-stats` - Match statistics
- `/overall-stats` - Overall player statistics
- `/opponents` - Opponent analysis
- `/settings` - User settings
- `/api/recordings` - Recordings list API
- `/api/recordings/[id]` - Single recording API (returns signed URLs for video + heatmaps)

## Development Commands

### Backend

**Install dependencies**:
```bash
cd backend
pip install -r ../requirements.txt
```

**Run pipeline locally** (for testing, requires weights):
```bash
# From project root
python -m backend.pipeline.run --video /path/to/video.mp4 --output output.mp4

# Note: This runs in local_mode (skips Supabase upload)
```

**Deploy to Modal**:
```bash
modal deploy backend/app.py
```

**Test Modal function locally**:
```bash
modal run backend/app.py
```

**Environment variables** (create `backend/.env`):
- `SUPABASE_URL` - Supabase project URL
- `SUPABASE_SERVICE_ROLE_KEY` - Supabase service role key

### Frontend

**Install dependencies**:
```bash
cd frontend/web
npm install
```

**Development server** (runs on http://localhost:3000):
```bash
cd frontend/web
npm run dev
```

**Production build**:
```bash
cd frontend/web
npm run build
npm start
```

**Lint**:
```bash
cd frontend/web
npm run lint
```

**Environment variables** (create `frontend/web/.env.local`):
- `NEXT_PUBLIC_SUPABASE_URL` - Supabase project URL
- `NEXT_PUBLIC_SUPABASE_ANON_KEY` - Supabase anonymous key

## Video Processing Pipeline

The pipeline processes tennis videos in two passes using a **streaming approach** (never loads all frames into memory):

1. **Pass 1: Ball + Player Tracking** (frames 0-N)
   - Ball detection on every frame using YOLO (TrackNet)
   - Player detection and tracking using YOLOv8x
   - Results stored in `ball_track` list and `player_detections` list
   - Only detection results stored (not frame pixels) - memory efficient!
   - Player filtering: Keeps only 2 main players closest to court
   - Bounce detection using CatBoost on ball trajectory
   - Progress: 5% → 50%


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AggieSportsAnalytics/CourtCheck](https://github.com/AggieSportsAnalytics/CourtCheck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
