---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

VoiceVault is an enterprise voice intelligence platform for hackathon submission (RAISE2025 - Vultr Track). It transforms voice conversations into actionable insights using AI/ML with enterprise-grade security.

**Core Pipeline**: Audio Upload → ASR Provider (Fast Transcription) → LLM Provider (Intelligent Analysis) → VoiceVault Dashboard → Enterprise Integrations

## System Architecture

### Multi-Service Architecture
The application uses a microservices architecture with 5 containerized services defined in `compose.yml`:

1. **UI Service** (`/ui`): React + TypeScript + Vite frontend served via Nginx on port 3000
2. **API Service** (`/api`): FastAPI backend on port 8000 with SQLAlchemy ORM and Alembic migrations
3. **Download Worker** (`/worker` with `WORKER_MODE=download`): Background service for URL-based content extraction using yt-dlp
4. **ASR Worker** (`/worker` with `WORKER_MODE=asr`): Background service for audio transcription using Groq Whisper
5. **Database**: PostgreSQL 17 with automatic table creation on startup
6. **MinIO**: S3-compatible object storage for development (production uses Vultr Object Storage)

### Worker Architecture Pattern
The `/worker` directory contains a unified worker codebase that runs in two modes:
- **Download Worker**: Polls database for entries with `entry_type='url'` and `status='NEW'`, downloads videos/audio using yt-dlp, extracts audio with FFmpeg, uploads to S3, updates status to `IN_PROGRESS`
- **ASR Worker**: Polls database for entries with `status='IN_PROGRESS'`, downloads audio from S3, transcribes using Groq Whisper API, stores transcript in database, updates status to `READY`

Both workers use the same Docker image but different environment variables to control behavior.

### Entry Status Workflow
Entries progress through these states:
- `NEW` → Entry just created, queued for download worker
- `IN_PROGRESS` → Audio downloaded, queued for ASR worker
- `READY` → Transcript ready, available for chat/analysis
- `COMPLETE` → User marked as finished (optional)

### Database Schema
The primary model is `Entry` in `/api/app/models/entry.py` and `/worker/app/models/entry.py`:
- Uses SQLAlchemy ORM with automatic table creation via `Base.metadata.create_all()` in `app/main.py:15`
- No manual migrations needed - tables auto-created on startup
- Database URL constructed from environment variables via `app/db/database.py`

## Development Commands

### Local Development Setup
```bash
# 1. Copy environment configuration
cp .env.example .env
# Edit .env with your API keys (GROQ_API_KEY is required)

# 2. Start all services with Docker Compose (includes PostgreSQL and MinIO)
docker compose up --build

# 3. Access services
# - Frontend: http://localhost:3000
# - API: http://localhost:8000
# - API Docs: http://localhost:8000/docs
# - PostgreSQL: localhost:5432
# - MinIO Console: http://localhost:9001
```

### Frontend Development (React + TypeScript + Vite)
```bash
cd ui

# Install dependencies
npm install

# Development server with hot reload
npm run dev

# Production build
npm run build

# Preview production build
npm preview
```

### Backend Development (FastAPI)
```bash
cd api

# Install dependencies
pip install -r requirements.txt

# Run API server directly (without Docker)
# Set DATABASE_URL environment variable first
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# Access API documentation
# Swagger UI: http://localhost:8000/docs
# ReDoc: http://localhost:8000/redoc
```

### Worker Development
```bash
cd worker

# Install dependencies
pip install -r requirements.txt

# Run download worker
WORKER_MODE=download DATABASE_URL=postgresql://... python -m app.main

# Run ASR worker
WORKER_MODE=asr DATABASE_URL=postgresql://... python -m app.main
```

### Database Operations
```bash
# The application uses automatic table creation on startup (app/main.py:15)
# No manual migrations required for development

# If using Alembic for schema changes:
cd api

# Create new migration
alembic revision --autogenerate -m "Description of changes"

# Apply migrations
alembic upgrade head

# Rollback migration
alembic downgrade -1

# View migration history
alembic history
```

### Production Deployment
```bash
# Production uses compose.prod.yml with self-contained containers
# (no volume mounts, code embedded in images)

# Option 1: Build locally and run
docker compose -f compose.prod.yml build
docker compose -f compose.prod.yml up -d

# Option 2: Build and push to registry, then pull on production server
export REGISTRY=fra.vultrcr.com/raise2025/
export VERSION=v1.0.0

# On build machine:
docker compose -f compose.prod.yml build
docker compose -f compose.prod.yml push

# On production server:
docker compose -f compose.prod.yml pull
docker compose -f compose.prod.yml up -d

# View logs
docker compose -f compose.prod.yml logs -f

# Stop services
docker compose -f compose.prod.yml down
```

### Testing and Debugging
```bash
# View service logs
docker compose logs -f api
docker compose logs -f worker-asr
docker compose logs -f worker-download

# Check service status
docker compose ps


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nezhar/voicevault](https://github.com/nezhar/voicevault) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
