---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ContextuAI Solo is a single-user desktop AI assistant with 96 pre-built business agents (108 in repo, engineering and coder-companion categories excluded from desktop's main library). It's a Tauri v2 desktop app with a React 19 frontend and a FastAPI Python backend running as a sidecar process. Data is stored locally in SQLite. Supports cloud providers (Anthropic, OpenAI, Google Gemini, AWS Bedrock, Ollama) and local GGUF models via llama-cpp-python. The shell has two top-level modes — **Solo** (business assistant) and **Coder** (local coding workspace) — toggled from a top-center pill in the title bar (`Cmd/Ctrl+Shift+M`).

## Commands

### Frontend (run from `frontend/`)
```bash
npm install                  # Install dependencies
npm run dev                  # Vite dev server on port 1420
npm run build                # TypeScript check + Vite production build
npm run tauri dev            # Run full Tauri desktop app (dev mode)
npm run tauri build          # Build platform installers (.exe/.msi)
npx playwright test          # Run E2E tests (requires dev server running)
npx playwright test tests/e2e/some-test.spec.ts  # Run a single test
```

### Backend (run from `backend/`)
```bash
pip install -r requirements.txt   # Install dependencies
CONTEXTUAI_MODE=desktop uvicorn app:app --host 127.0.0.1 --port 18741 --reload  # Dev server
pytest tests/ -v                  # Run backend tests
```

### Quick Start
```bash
./run.sh                     # Creates venv, installs deps, starts backend
# Then in another terminal: cd frontend && npm run dev
```

### Docker
```bash
docker compose up            # Starts backend only; run frontend separately
```

## Architecture

### Three-Layer Desktop App
```
Tauri Shell (Rust)  →  React SPA (port 1420)  →  FastAPI Sidecar (port 18741)
```

- **Tauri shell** (`frontend/src-tauri/src/`): Window management, system tray, sidecar process lifecycle. `main.rs` boots the app, `sidecar.rs` spawns/health-checks the backend, `commands.rs` provides IPC handlers.
- **React frontend** (`frontend/src/`): Routes in `routes/`, reusable UI in `components/ui/`, API clients in `lib/api/`, transport layer in `lib/transport.ts`.
- **FastAPI backend** (`backend/`): Entry point is `app.py`. Routers in `routers/`, business logic in `services/`, data access in `repositories/`, adapters in `adapters/`.

### Frontend-Backend Communication
- **Tauri mode**: React → Tauri IPC (`api_request` command) → HTTP to sidecar
- **Dev mode**: React → direct HTTP fetch to `http://127.0.0.1:18741/api/v1`
- **Streaming**: Always SSE over HTTP (not IPC), consumed via `streamRequest()` in `transport.ts`. Supports `AbortSignal` for cancelling mid-stream.
- Retry logic: 5 attempts with exponential backoff
- **Stream abort**: Frontend uses `AbortController` to cancel HTTP fetch on stop. Backend `LocalModelService` has `asyncio.Lock` to prevent concurrent model access — second request waits for first to finish/abort.

### Database: SQLite with MongoDB-Compatible API
The backend was ported from MongoDB. A compatibility layer preserves the Motor API:
- `adapters/sqlite_adapter.py` — Async SQLite wrapper (aiosqlite)
- `adapters/motor_compat.py` — `DatabaseProxy`/`CollectionProxy` that mimic Motor's `AsyncIOMotorDatabase`/`AsyncIOMotorCollection`
- Tables use `_id TEXT PRIMARY KEY, data JSON NOT NULL` pattern — documents stored as JSON blobs
- MongoDB query operators (`$set`, `$in`, `$regex`, `$gt`, etc.) are translated to SQL/JSON expressions
- DB path: `~/.contextuai-solo/data/contextuai.db`

### Backend Patterns
- **Repository pattern**: `repositories/*.py` — Generic async CRUD via `BaseRepository<T>`
- **Service layer**: `services/*.py` — Business logic injected with repositories via FastAPI `Depends()`
- **Workspace orchestration**: `services/workspace/orchestrator.py` polls job queue, `agent_runner.py` executes individual agents, `checkpoint_service.py` handles pause/resume
- **Crew system**: Multi-agent teams with persistent memory (`crew_memory_service.py`)

### Local AI Models (`backend/routers/local_models.py`)
GGUF models downloaded from HuggingFace, stored in `~/.contextuai-solo/models/`. Inference via llama-cpp-python, with automatic GPU offload (Apple Metal / NVIDIA CUDA / Vulkan) when the installed build supports it, falling back to CPU. Offload is gated by `llama_cpp.llama_supports_gpu_offload()` and tunable via the `LOCAL_MODEL_GPU_LAYERS` env var (`auto` (default) / `0` to force CPU / `<N>` layers); the loaded model stays resident in RAM across messages and only reloads on a model swap.
- 41 curated GGUF models (Gemma 4, Qwen 3/3.5, DeepSeek R1, Llama 3, Mistral, Phi-4, etc.) from 0.5B to 70B
- Download: `POST /api/v1/local-models/{model_id}/download` (SSE progress)
- Sync to DB: `POST /api/v1/local-models/sync` (registers downloaded models in the `models` collection)
- Models appear in chat dropdown after sync

### Agent Library (`agent-library/`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [contextuai/contextuai-solo](https://github.com/contextuai/contextuai-solo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
