---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

VoiceLinkVR Server is a Python-based voice transcription and translation service that provides:
- Speech recognition using Whisper and SenseVoice
- Multi-language translation (LibreTranslate + online services)
- User management and authentication (JWT-based)
- Rate limiting and request logging
- Web management interface

**Architecture**: Modern FastAPI application (migrated from Flask) with async support, dependency injection, and automatic API documentation.

## Development Commands

### Local Development
```bash
# Install dependencies
pip install -r src/requirements.txt

# Run development server with hot reload
python run.py

# Run with custom workers (production mode)
UVICORN_WORKERS=4 python run.py

# Run with specific reload setting
UVICORN_RELOAD=true python run.py
```

### Testing
```bash
# Test translation functionality
python test_translate.py
```

### Docker Deployment
```bash
# CPU-only deployment
docker-compose -f docker-compose-cpu.yml up -d

# CUDA deployment (requires CUDA 12.2+)
docker-compose -f docker-compose-cuda.yml up -d

# China mirror deployments
docker-compose -f docker-compose-cpu-cn.yml up -d
docker-compose -f docker-compose-cuda-cn.yml up -d
```

### Production Server
```bash
# Direct uvicorn command
uvicorn src.main:app --host 0.0.0.0 --port 8980 --workers 4
```

## Architecture & Code Structure

### Core Architecture Pattern
- **Framework**: FastAPI with async/await support
- **Database**: SQLAlchemy ORM with SQLite/MySQL support
- **Authentication**: JWT tokens using python-jose
- **Password Hashing**: Werkzeug (bcrypt)
- **Rate Limiting**: slowapi with Redis backend
- **Template Engine**: Jinja2
- **Server**: Uvicorn ASGI server

### Directory Structure
```
src/
├── core/           # Core functionality
│   ├── config.py   # Pydantic settings management
│   ├── dependencies.py # FastAPI dependency injection
│   ├── services.py # Business logic (translation, audio processing)
│   ├── rate_limiter.py  # Rate limiting with Redis/memory backend
│   ├── logging_config.py # Logging configuration
│   ├── text_compressor.py # Text compression utilities
│   └── translation_service.py # Translation service abstraction
├── db/             # Database layer
│   ├── base.py     # SQLAlchemy base configuration
│   └── models.py   # ORM models (User, RequestLog)
├── routers/        # API routes
│   ├── api.py      # Main API endpoints (/api/*)
│   ├── manage_api.py # Admin API endpoints (/manageapi/*)
│   └── ui.py       # Web interface routes (/ui/*)
├── schemas/        # Pydantic data models
├── templates/      # HTML templates
├── static/         # Static files
├── main.py         # FastAPI app entry point
└── requirements.txt # Python dependencies
```

### Key Dependencies
- **fastapi**: Modern web framework
- **uvicorn[standard]**: ASGI server
- **sqlalchemy**: ORM and database toolkit
- **python-jose[cryptography]**: JWT implementation
- **slowapi**: Rate limiting
- **httpx**: Async HTTP client
- **translators**: Translation services integration
- **openai**: OpenAI client for Whisper API
- **apscheduler**: Background task scheduling

### External Service Dependencies
- **Whisper Service**: Speech recognition (default: localhost:8000)
- **SenseVoice Service**: Alternative speech recognition (default: localhost:8800)
- **LibreTranslate**: Local translation service (default: localhost:5000)
- **Redis**: Rate limiting backend (optional)

## Key Implementation Details

### Configuration Management
All configuration is managed through `src/core/config.py` using Pydantic Settings:
- Environment variables take precedence over defaults
- `.env` file support for local development
- Service URLs, API keys, database paths, rate limiting settings

### Authentication Flow
1. Users login via `/api/login` with username/password
2. Server returns JWT access token (7-day expiry)
3. Subsequent API calls include token in Authorization header
4. Admin users have `is_admin=True` flag in database

### Rate Limiting
- Global rate limiting via slowapi middleware
- Per-user rate limiting based on `limit_rule` field
- Redis backend for distributed rate limiting
- Default limits: "10000/day;1000/hour"

### Request Logging
- All API requests automatically logged to `request_log` table
- Includes username, IP, endpoint, duration, status
- Middleware-based implementation in `src/main.py`

### Translation Services
1. **Local LibreTranslate**: Primary translation service
2. **Online Translators**: Fallback via `translators` library
3. **Service Priority**: Configurable via `TRANSLATOR_SERVICES_LIST`
4. **Timeout Handling**: 1.5s timeout with failover

### Audio Processing
- Supports WAV and OPUS audio formats
- OPUS decoding via `opuslib`
- Automatic format detection and conversion
- Content filtering for error results
- Text compression for repeated characters (configurable)

### Scheduled Tasks
- **User Expiration Check**: Daily at midnight UTC, disables expired users
- **Filter Config Update**: Weekly on Monday at 3:00 AM UTC, updates filter rules from web

## Common Development Tasks

### Adding New API Endpoints

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VoiceLinkVR/VoiceLinkServer](https://github.com/VoiceLinkVR/VoiceLinkServer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
