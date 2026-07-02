---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

WebATM is a modern web client for the BlueSky Air Traffic Management (ATM) simulator. It provides a web interface using MapLibre GL for aircraft visualization and radar display, connecting to BlueSky simulation servers via network protocols. The project features Docker containerization, TypeScript-based frontend, and connection to a BlueSky server.

### Build Variants

WebATM ships in two variants, both built from this repo:

- **Standalone (`webatm`)** — the default. Web client only; connects to a BlueSky server you run yourself.
- **Integrated (`webatm-integrated`)** — bundles BlueSky in the same container and adds in-app server lifecycle controls (Start/Stop/Restart/Kill) plus a live server-log tab. File management auto-wires to BlueSky's working directory.

The integrated variant is **fully opt-in and gated end-to-end**:
- **Backend**: `WebATM/app.py` calls `webatm_integrated.register()` only when `WEBATM_INTEGRATED=1`. Any failure is logged and non-fatal.
- **Frontend**: `frontend/src/main.ts` dynamic-imports `./integrated/index` only when the `INTEGRATED_BUILD` compile-time constant (webpack `DefinePlugin`) is `true`. The `integrated` chunk is dead-code-eliminated from the default bundle.
- **Dependency direction**: the core `WebATM` package never imports `webatm_integrated`. The arrow only points integrated → core.

When working on integrated features, **do not** add imports from core into integrated code paths or vice versa beyond the one registration hook.

## Requirements

- **Python**: 3.13 or higher
- **Node.js**: 22+ with npm (for TypeScript development)
- **Docker**: 20.10+ with Docker Compose 2.0+ (for containerized deployment)
- **BlueSky**: Version 1.1.0 or higher (automatically managed by WebATM)

## Running and Testing Commands

### Starting the Application
```bash
# Run the main web server
python WebATM.py
```

The web interface will be available at http://localhost:8082

### Python Environment and Tests
Dependencies are managed with [uv](https://docs.astral.sh/uv/) from `pyproject.toml` and the pinned `uv.lock` (the legacy `requirements*.txt` files have been removed).

```bash
# Sync the dev environment (core deps + PEP 735 `dev` group)
uv sync

# Sync with production extras (gunicorn, eventlet)
uv sync --extra prod

# Run the test suite (collects both core and integrated suites)
uv run pytest                # everything (core + integrated)
uv run pytest -m core        # core `webatm` package only (tests/)
uv run pytest -m integrated  # optional `webatm_integrated` package only
```

### Frontend (TypeScript) Development
The TypeScript source lives in the top-level `frontend/` directory. Build output is emitted to `WebATM/static/dist/` (consumed by the Flask templates) and vendored third-party assets (FontAwesome, MapLibre GL CSS) are copied into `WebATM/static/vendor/` by the `vendor-assets` prebuild step.

```bash
# Navigate to frontend directory
cd frontend/

# Install dependencies
npm install

# Production build (also runs vendor-assets prebuild)
npm run build              # alias for build:production
npm run build:production   # NODE_ENV=production webpack
npm run build:integrated   # INTEGRATED_BUILD=true webpack (emits integrated chunk)
npm run build:dev          # development build

# Watch for changes during development
npm run watch
npm run watch:integrated   # watch with integrated chunk emitted

# Type checking only
npm run type-check

# Copy vendored assets only
npm run vendor-assets
```

A convenience script `script/build_frontend.sh` runs the full production build from the repo root. Pass `--integrated` to build the integrated bundle instead.

### Docker Deployment

#### Using Docker Compose (Recommended)
```bash
# Start the application stack
docker-compose up -d

# View logs
docker-compose logs -f webatm

# Stop the stack
docker-compose down
```

#### Building Docker Image Manually
```bash
# Standalone image
docker build -t webatm:latest .
docker run -p 8082:8082 webatm:latest

# Integrated image (bundles BlueSky from amvlab/bluesky)
docker build -f Dockerfile.integrated -t webatm-integrated:latest .
docker run -p 8082:8082 webatm-integrated:latest
```

The integrated image bakes in `WEBATM_INTEGRATED=1`, `BLUESKY_SERVER_HOST=localhost`, `WEB_HOST=0.0.0.0`, and `WEB_PORT=8082`, and runs under gunicorn with the `gthread` worker (not eventlet — incompatible with the blocking subprocess pipe the integrated build reads in a background thread). The standalone image is unaffected by anything in `WebATM-integrated/`.

#### Docker Environment Variables
- `FLASK_ENV` - Set to 'production' for production deployment
- `BLUESKY_SERVER_HOST` - BlueSky server hostname/IP address (default: localhost)
- `WEB_PORT` - Web server port (default: 8082)
- `WEB_HOST` - Web server bind address (default: localhost for security, use 0.0.0.0 for Docker)
- `HEARTBEAT_INTERVAL` - Heartbeat interval in seconds (default: 30)
- `WEBATM_INTEGRATED` - Set to `1` to enable the integrated extensions (server lifecycle + log streaming). Off by default; ignored if `webatm_integrated` isn't installed.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amvlab/WebATM](https://github.com/amvlab/WebATM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
