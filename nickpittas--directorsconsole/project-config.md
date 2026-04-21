---
trigger: always_on
description: **Director's Console** is a unified AI VFX production pipeline (Project Eliot) that combines specialized tools into a cohesive ecosystem for high-end AI-generated video and image production.
---

# AGENTS.md - Development Guidelines for Director's Console

## Project Overview

**Director's Console** is a unified AI VFX production pipeline (Project Eliot) that combines specialized tools into a cohesive ecosystem for high-end AI-generated video and image production.

### The Components

1. **CinemaPromptEngineering (CPE)** - Cinematic rules engine for camera, lighting, and era-specific prompt generation
   - FastAPI backend with Pydantic models (`api/main.py`)
   - React/TypeScript frontend with Vite build system
   - Supports 35+ live-action film presets and animation presets
   - Integrates with multiple LLM providers (OpenAI, Anthropic, DeepInfra, etc.)
   - OAuth flow implementation for provider authentication
   - Templates system for ComfyUI workflow management
   - **⚠️ NOTE:** `ComfyCinemaPrompting/cinema_rules/` is a DUPLICATE - use main `cinema_rules/` module

2. **Orchestrator** - Distributed render farm manager for ComfyUI
   - FastAPI REST API for job submission (`orchestrator/api/server.py`)
   - Multi-backend ComfyUI node management
   - Real-time job progress via WebSocket
   - Gallery API: 23 endpoints for file browsing, batch rename, ratings, tags, search, trash
   - **⚠️ NOTE:** Frontend communicates DIRECTLY with ComfyUI nodes, NOT through Orchestrator API for workflow execution
   - Orchestrator used for: job groups, backend status, project management, gallery operations
   - Job groups feature for parallel execution across multiple backends

3. **Gallery** - Full-featured media browser and file management (top-level tab)
   - React/TypeScript frontend module (`frontend/src/gallery/`)
   - 37 frontend files: main UI, Zustand store, API service, 28 components
   - Backend: 23 REST endpoints on Orchestrator (`/api/gallery/*`) via `gallery_routes.py`
   - Storage: JSON flat-file at `{projectPath}/.gallery/gallery.json` (NAS/CIFS compatible — SQLite incompatible with CIFS mounts)
   - Cross-tab communication with Storyboard via `window` CustomEvents
   - **⚠️ NOTE:** Gallery metadata is per-project, stored alongside project files on NAS

### Communication Architecture

- **Inter-Process**: JSON manifests via shared storage (TrueNAS) + REST API status broadcasting
- **Data Flow**: CPE → Orchestrator → ComfyUI Render Nodes
- **Gallery ↔ Storyboard**: Cross-tab CustomEvents on `window` (reference images, workflow restore, file rename sync)
- **Gallery → Orchestrator**: REST API for all gallery operations (`/api/gallery/*`)
- **Storage Location**: User based, set from Project Settings in the Frontend

---

## Technology Stack

### Backend
- **Language**: Python 3.10+
- **Framework**: FastAPI (for API layers)
- **Data Validation**: Pydantic v2
- **Async**: asyncio, httpx, aiohttp
- **Process Management**: watchdog (for file system monitoring)
- **Packaging**: PyInstaller for standalone executables

### Frontend (CPE)
- **Framework**: React 18 with TypeScript
- **Build Tool**: Vite 5
- **State Management**: Zustand
- **Data Fetching**: TanStack Query (React Query) v5
- **UI Components**: React Select, Lucide React icons
- **Styling**: Emotion (CSS-in-JS)

### Testing
- **Framework**: pytest
- **HTTP Testing**: fastapi.testclient.TestClient
- **Location**: `/tests` at project root

### Logging
- **Primary**: loguru
- **Fallback**: Standard logging module

---

## Project Structure

```
DirectorsConsole/
├── CinemaPromptEngineering/        # CPE module
│   ├── api/                        # FastAPI application
│   │   ├── main.py                 # Main API with endpoints
│   │   ├── templates.py            # Template router
│   │   └── providers/              # LLM provider integrations
│   ├── cinema_rules/               # Core rules engine
│   ├── frontend/                   # React frontend
│   │   └── src/
│   │       ├── storyboard/         # Storyboard Canvas tab
│   │       └── gallery/            # Gallery tab (37 files)
│   │           ├── GalleryUI.tsx    # Main gallery component
│   │           ├── store/           # Zustand state store
│   │           ├── services/        # API client service
│   │           └── components/      # 28 UI components
│   ├── ComfyCinemaPrompting/       # ComfyUI custom node
│   └── Documentation/              # Comprehensive docs
│
├── Orchestrator/                   # Render farm module
│   └── orchestrator/
│       ├── api/                    # FastAPI server package
│       │   ├── __init__.py         # Exposes app from server.py
│       │   ├── server.py           # FastAPI server (main entry)
│       │   └── gallery_routes.py   # Gallery API router (23 endpoints)
│       ├── gallery_db.py           # JSON flat-file gallery storage
│       ├── app.py                  # Desktop app entry
│       ├── main.py                 # CLI entry
│       ├── core/                   # Core engine
│       ├── backends/               # ComfyUI backend management
│       └── storage/                # Database and repositories
│
├── tests/                          # Cross-module tests
├── docs/                           # Additional documentation
├── WorkflowsForImport/             # ComfyUI workflow templates

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NickPittas/DirectorsConsole](https://github.com/NickPittas/DirectorsConsole) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
