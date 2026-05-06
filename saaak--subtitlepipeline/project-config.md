---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SubtitlePipeline is a self-hosted subtitle generation pipeline that automatically generates Chinese subtitles for video files. It's designed for media server integration (Jellyfin/Emby/Plex) and supports multiple ASR providers.

**Tech Stack:**
- Backend: Python 3.12, FastAPI, SQLite (WAL mode), Pydantic
- Frontend: React 18, TypeScript, React Router 6, Vite 5
- ASR: WhisperX, Faster-Whisper, Anime-Whisper, Qwen-ASR
- Deployment: Docker, Supervisor (multi-process orchestration)

## Development Commands

### Backend

```bash
cd backend

# Install dependencies
pip install -r requirements.txt

# Run individual processes (for local development)
python -m app.api_server      # FastAPI server on port 8000
python -m app.scanner_process # File scanner daemon
python -m app.worker_process  # Task processor daemon

# Run tests
python -m unittest backend/tests/test_mvp.py
```

### Frontend

```bash
cd frontend

# Install dependencies
npm install

# Development server (with hot reload)
npm run dev

# Production build
npm run build
```

### Docker

```bash
# CPU version
docker compose up --build -d

# GPU version (NVIDIA CUDA)
docker compose -f docker-compose.gpu.yml up --build -d

# View logs
docker compose logs -f

# Stop services
docker compose down
```

## Architecture

### Three-Process Model

The application runs three independent processes orchestrated by Supervisor:

1. **API Server** (`app/api_server.py`)
   - FastAPI application serving REST API and frontend SPA
   - Handles configuration, task management, model downloads
   - Entry point: `app/main.py:create_app()`

2. **Scanner** (`app/scanner_process.py`)
   - Periodically scans `input_dir` for new video files
   - Creates pending tasks in SQLite for discovered videos
   - Skips files in `.subpipeline` directories and muxed outputs
   - Service: `app/runtime.py:ScannerService`

3. **Worker** (`app/worker_process.py`)
   - Processes queued tasks through the subtitle pipeline
   - Handles ASR, translation, subtitle rendering, and optional muxing
   - Supports resume from intermediate checkpoints on retry
   - Service: `app/runtime.py:WorkerService`

All three processes share a single SQLite database with WAL mode enabled for concurrent access.

### Core Pipeline Flow

The subtitle generation pipeline (`app/pipeline.py`) executes these stages:

1. **extract_audio** - Extract audio track from video using FFmpeg
2. **run_asr** - Transcribe audio using selected ASR provider (WhisperX/Faster-Whisper/Anime-Whisper/Qwen)
3. **process_text_segments** - Clean and normalize transcribed text
4. **translate_segments** - Translate via OpenAI-compatible API (if enabled)
5. **render_srt** - Generate `.srt` subtitle files (bilingual or monolingual)
6. **mux_subtitle** - Optionally mux subtitles back into video container

Each stage saves intermediate results to `work_dir` for resume capability. The worker checks `cancel_requested` flag between stages.

### Multi-Provider ASR System

The `app/model_manager.py` manages ASR models across four providers:

- **WhisperX** (default): CTranslate2 + forced alignment for accurate timestamps
- **Faster-Whisper**: Lightweight CTranslate2 inference, faster startup
- **Anime-Whisper**: Japanese anime dialogue optimization
- **Qwen-ASR**: Multilingual transformer-based model with language detection

**Model Naming Convention:**
- Models use provider-prefixed names: `whisperx-small`, `faster-whisper-large-v3`, `anime-whisper`, `qwen3-asr-1.7b`
- Legacy aliases (e.g., `small` → `whisperx-small`) are supported for backward compatibility
- When activating a model, both `whisper.model_name` and `whisper.provider` are updated atomically

**Provider-Specific Configuration:**
Each provider has its own config section (`whisperx`, `faster_whisper`, `anime_whisper`, `qwen`) with provider-specific tuning options (VAD, alignment, temperature, etc.).

### Database Layer

`app/store.py` provides the `Database` class with these responsibilities:

- **Configuration Management**: Hierarchical config stored as JSON in `system_config` table
- **Task Lifecycle**: Create, update, retry, cancel tasks with status tracking
- **Logging**: Per-task structured logs with pagination
- **Resume Logic**: Tracks completed stages and intermediate file paths for resume-from-checkpoint

**Important:** The database uses SQLite with `PRAGMA journal_mode=WAL` for concurrent reads/writes across the three processes.

### Frontend Structure

React SPA with client-side routing (`frontend/src/App.tsx`):

- **SetupWizard** - First-run configuration flow (input directory, ASR model, translation API)
- **TasksPage** - Task list with status filtering and pagination
- **TaskDetailPage** - Task details, logs, retry/cancel actions
- **ModelManagerPage** - Download, activate, delete ASR models
- **SettingsPage** - Configuration editor for all settings groups

API client (`frontend/src/api.ts`) provides typed fetch wrappers for all backend endpoints.

## Configuration

Configuration is stored in SQLite (`system_config` table) and managed via `/api/config` endpoints. Key groups:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [saaak/SubtitlePipeline](https://github.com/saaak/SubtitlePipeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
