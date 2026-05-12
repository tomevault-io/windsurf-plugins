---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

YA-WAMF (Yet Another WhosAtMyFeeder) is a bird classification system that integrates with Frigate NVR to identify birds visiting feeders using machine learning. The system receives MQTT events from Frigate, classifies bird species using local ML models (TFLite/ONNX), correlates with audio detections from BirdNET-Go, and provides a real-time web dashboard with notifications.

**Tech Stack:**
- Backend: Python 3.12 + FastAPI + SQLite + Alembic
- Frontend: Svelte 5 + TypeScript + Tailwind CSS + Vite
- ML: ONNX Runtime / TensorFlow Lite (MobileNetV2, ConvNeXt, EVA-02)
- Messaging: MQTT (aiomqtt) for Frigate events, SSE for frontend updates
- Deployment: Docker Compose with separate backend/frontend containers

## Git Commit Rules

- **Never** add `Co-Authored-By:`, `Co-authored-by:`, or any AI attribution trailer to commit messages.
- **Never** reference Claude, Gemini, or any AI tool in commit messages, PR descriptions, or issue comments.
- Commit messages should read as if written by the project owner.

## Development Commands

### Backend

```bash
cd backend

# Setup
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt

# Run development server (with auto-reload)
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# Database migrations
alembic revision --autogenerate -m "description"
alembic upgrade head

# Testing
pytest                                    # Run all tests
pytest --cov=app --cov-report=html        # With coverage
pytest tests/test_proxy.py -v             # Specific file
pytest tests/test_proxy.py::test_name -v  # Specific test

# Code quality
ruff check .                              # Linting
```

### Frontend

```bash
cd apps/ui

# Setup
npm install

# Development server (proxies API to localhost:8000)
npm run dev

# Build for production
npm run build

# Type checking
npm run check
```

### Docker

```bash
# Full stack
docker compose up -d
docker compose logs -f
docker compose logs -f yawamf-backend

# Rebuild after changes
docker compose build
docker compose up -d

# Development mode (with hot reload)
docker compose -f docker-compose.dev.yml up -d
```

## Architecture

### Data Flow

```
Frigate (MQTT) → MQTTService → EventProcessor → ClassifierService → DetectionRepository → SQLite
                                     ↓                                      ↓
                              FrigateClient (snapshot)              Broadcaster (SSE)
                              WeatherService                               ↓
                              AudioService (BirdNET)                  Frontend (realtime)
                              TaxonomyService
                              NotificationService
```

### Key Components

**Backend Services** (`backend/app/services/`):
- `mqtt_service.py` - Subscribes to `frigate/events` MQTT topic
- `event_processor.py` - Orchestrates detection pipeline (200+ lines, needs refactoring)
- `classifier_service.py` - ML inference engine (ONNX/TFLite model loading and prediction)
- `frigate_client.py` - Fetches snapshots and clips from Frigate API
- `audio/audio_service.py` - Correlates visual detections with BirdNET-Go audio buffer
- `taxonomy/taxonomy_service.py` - iNaturalist-based scientific ↔ common name mapping
- `ai_service.py` - LLM integration (Gemini/OpenAI) for behavioral analysis
- `broadcaster.py` - Server-Sent Events (SSE) for real-time frontend updates
- `notification_service.py` - Discord, Telegram, Pushover notifications
- `auto_video_classifier_service.py` - Background video frame analysis for higher accuracy
- `model_manager.py` - Model download and management
- `telemetry_service.py` - Anonymous usage metrics (opt-in)

**Backend Routers** (`backend/app/routers/`):
- `events.py` - Detection CRUD, filtering, pagination, deletion
- `species.py` - Species aggregation, statistics, counts
- `settings.py` - Configuration management (read/update config.json)
- `proxy.py` - Frigate media proxy (thumbnails, snapshots, clips with HTTP Range support)
- `stream.py` - SSE endpoint for real-time updates
- `classifier.py` - Model status, health checks
- `ai.py` - LLM behavioral analysis endpoint
- `audio.py` - Recent BirdNET audio detections
- `backfill.py` - Historical Frigate event reprocessing
- `stats.py` - Analytics and metrics
- `models.py` - Model management endpoints

**Frontend Pages** (`apps/ui/src/lib/pages/`):
- `Dashboard.svelte` - Real-time detection grid with SSE
- `Events.svelte` - Paginated detection list with filters
- `Species.svelte` - Species statistics and taxonomy info
- `Settings.svelte` - Configuration UI (camera selection, thresholds, notifications, etc.)

**Frontend Components** (`apps/ui/src/lib/components/`):
- `DetectionCard.svelte` - Individual detection display with video playback
- `VideoPlayer.svelte` - Proxied Frigate clip player with seeking support
- `SpeciesDetailModal.svelte` - Species info modal with LLM analysis
- `Header.svelte` - Navigation and dark mode toggle
- `Footer.svelte` - Footer with version and links

### Database Schema

**Primary Table:** `detections`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jellman86/YetAnother-WhosAtMyFeeder](https://github.com/Jellman86/YetAnother-WhosAtMyFeeder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
