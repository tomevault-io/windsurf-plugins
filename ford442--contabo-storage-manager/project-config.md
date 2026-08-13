---
trigger: always_on
description: <!-- From: /root/contabo_storage_manager/AGENTS.md -->
---

<!-- From: /root/contabo_storage_manager/AGENTS.md -->
<!-- From: /root/contabo_storage_manager/AGENTS.md -->
# AGENTS.md – Contabo Storage Manager

> This file contains essential context for AI coding agents working on this project.  
> The project is a multi-service storage bridge and static file server for a Contabo VPS.

---

## Project Overview

This project provides webhook receivers, REST APIs, and static file serving for multiple frontend applications. It persists payloads and files locally under a single directory, syncs to external FTP/SFTP (including automatic FLAC mirroring), and serves content back over HTTPS via Nginx.

### Architecture

```
Internet ──→ (nginx / Caddy / direct)
               │
               ├─── :8000  Python Bridge (FastAPI) ── primary service
               │             ├── Webhooks  → /webhook/*
               │             ├── Admin     → /admin (upload dashboard)
               │             ├── Shaders   → /api/shaders*, /api/maps
               │             ├── Music     → /api/songs*, /api/music/*
               │             ├── Sequencer → /api/songs*, /api/patterns*, /api/banks*, /api/samples*, /api/items
               │             ├── Notes     → /api/notes/*
               │             ├── Pachinball → /maps*, /music*, /backbox, /zones*, /upload/*
               │             ├── Leaderboard → /api/leaderboard*
               │             ├── Adventure  → /api/adventure/*
               │             ├── VPS Browser → /api/vps/*
               │             ├── Models    → /models/* (Range/HEAD support for WebLLM)
               │             ├── Mods      → /api/mods/* (MOD music files)
               │             ├── Presets   → /api/presets/* (MilkDrop .milk files)
               │             ├── Textures  → /api/textures/* (image textures)
               │             ├── Static    → /files/{path}
               │             └── Remote    → /api/admin/run, /api/admin/logs/{task_id}
               │
               ├─── :3000  Node Bridge (Express) ── minimal webhook receiver
               │             ├── POST /webhook/generic
               │             ├── POST /webhook/shopify
               │             └── POST /webhook/github
               │
               └─── :8080  Nginx static server (nginx-files container)
                             └── GET /<any-path>  → serves FILES_DIR directly

All services write to /home/ftpbridge/files  ←── single FTP account, vsftpd served
```

### Admin Dashboard

The Python bridge serves a universal upload dashboard at `GET /admin`. It supports drag-and-drop uploads for:
- Audio (`.mp3`, `.flac`, `.wav`, `.ogg`, `.m4a`, `.aac`) → `/api/songs/upload`
- Notes (`.md`) → `/api/notes/write/{name}`
- Shaders (`.wgsl`, `.json`) → `/api/shaders`

The admin panel also supports remote SSH command execution via `POST /api/admin/run` (whitelisted commands only) and SSE log streaming via `GET /api/admin/logs/{task_id}`.

### Officially Supported Apps

The following applications have **first-class support** with dedicated endpoints and organized storage:

1. **image_video_effects** — `/webhook/image-effects` — shader effects and outputs
2. **flac_player** — `/api/songs/*`, `/api/music/*` — audio library streaming
3. **web_sequencer** — `/webhook/sequencer`, `/api/songs/*`, `/api/patterns/*`, etc. — music composition
4. **rain_edit** — `/api/notes/*` — markdown note-taking (REST API only)
5. **cloud_notes** — `/api/notes/*`, `/webhook/notes` — cloud-based note sync with webhook support

All apps share the same single FTP account and storage directory (`/home/ftpbridge/files`). The **Notes** API is shared between `rain_edit` and `cloud_notes`, allowing both apps to read/write notes interchangeably.

---

## Technology Stack

### Python Bridge
- **Runtime**: Python 3.12+
- **Framework**: FastAPI 0.111+
- **Server**: Uvicorn with standard workers (default 2)
- **Key Dependencies**:
  - `fastapi` / `uvicorn[standard]` — web framework and ASGI server
  - `pydantic` / `pydantic-settings` — configuration and request/response models
  - `aiofiles` — async file I/O
  - `httpx` — async HTTP client
  - `python-multipart` — multipart form parsing
  - `paramiko` — SFTP connections
  - `asyncssh` — remote SSH command execution for admin panel
  - `jinja2` — admin panel HTML templating
  - `google-cloud-storage` — GCS bucket sync for music
  - `watchdog` — filesystem watcher for auto-indexing audio files
  - `pydub` — audio metadata extraction
  - `aiocache` — caching layer
  - `gunicorn` — production WSGI/ASGI worker alternative

### Node Bridge
- **Runtime**: Node.js 18+ (Node 20 recommended)
- **Framework**: Express.js 4.19+
- **Key Dependencies**:
  - `basic-ftp` — FTP operations
  - `winston` — structured logging
  - `express-rate-limit` — rate limiting on webhook endpoints
  - `dotenv` — environment configuration

### Infrastructure
- **Containerization**: Docker + Docker Compose (profiles: `full`, `python`, `node`, `storage`)
- **Static File Server**: Nginx (port 8080)
- **Deployment Target**: Contabo Ubuntu VPS with vsftpd
- **CI/CD**: GitHub Actions workflow (`.github/workflows/deploy.yml`) deploys on every push to `main`

---

## Project Structure

```
contabo_storage_manager/
├── packages/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ford442/contabo_storage_manager](https://github.com/ford442/contabo_storage_manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
