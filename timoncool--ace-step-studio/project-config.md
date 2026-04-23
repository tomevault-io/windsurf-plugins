---
trigger: always_on
description: ACE-Step Studio is a portable, self-contained AI music generation application. It bundles Python (with CUDA/PyTorch), Node.js, and the ACE-Step 1.5 ML pipeline into a single-terminal launcher. No installation required — extract and run.
---

# ACE-Step Studio — Agent Guidelines

## Project Overview

ACE-Step Studio is a portable, self-contained AI music generation application. It bundles Python (with CUDA/PyTorch), Node.js, and the ACE-Step 1.5 ML pipeline into a single-terminal launcher. No installation required — extract and run.

**GitHub:** https://github.com/timoncool/ACE-Step-Studio
**Upstream:** Forked from fspecii/ace-step-ui, heavily modified.

## Architecture

```
ACE-Step-Studio/
├── run.bat                    # Single-terminal launcher (prod)
├── run-dev.bat                # 3-terminal dev mode with Vite HMR
├── install.bat                # First-time setup
├── run-no-lm.bat              # Launch without LM (more VRAM)
├── python/                    # Portable Python 3.12 + CUDA 12.8
├── node/                      # Portable Node.js
├── ffmpeg/                    # Portable FFmpeg
├── models/                    # HuggingFace model cache
├── ACE-Step-1.5/              # ML pipeline (Gradio + FastAPI on port 8001)
│   ├── acestep/               # Python package
│   │   ├── acestep_v15_pipeline.py  # Main entry point
│   │   ├── llm_inference.py         # 5Hz LM (vLLM/nanovllm)
│   │   ├── api/http/                # FastAPI routes (/v1/init, /v1/models, /health)
│   │   └── ui/gradio/api/           # Gradio API routes
│   └── checkpoints/           # DiT + LM model weights
└── app/                       # Web application
    ├── index.html             # Entry point (uses CDN for Tailwind + esm.sh)
    ├── App.tsx                # Root component, generation logic
    ├── components/            # React components
    │   ├── CreatePanel.tsx    # Generation UI (Simple + Custom modes)
    │   ├── SongList.tsx       # Song list with model badges + generation time
    │   ├── RightSidebar.tsx   # Song details panel
    │   ├── Sidebar.tsx        # Left sidebar (GPU stats, navigation)
    │   └── ...
    ├── services/api.ts        # API client + Song data transformation
    ├── context/I18nContext.tsx # i18n provider
    ├── i18n/                  # Translations (en, ru, zh, ja, ko)
    ├── types.ts               # TypeScript interfaces
    ├── dist/                  # Vite production build (served by Express)
    └── server/                # Express backend (port 3001)
        ├── src/index.ts       # Server entry + static serving + pipeline manager
        ├── src/config/        # Config (ports, paths, pipeline settings)
        ├── src/routes/        # API routes (songs, generate, pipeline, etc.)
        ├── src/services/
        │   ├── pipeline-manager.ts  # Spawns/monitors/restarts Python process
        │   ├── acestep.ts           # Gradio client, job queue, generation logic
        │   ├── gradio-client.ts     # Gradio connection management
        │   └── storage/             # Local file storage
        └── src/db/            # SQLite migrations + connection pool
```

## Three-Process Architecture (Single Terminal)

Express is the supervisor. One `run.bat` → one terminal window:

1. **Express** (port 3001) — API server + serves Vite build as static files
2. **Python/Gradio** (port 8001) — ML pipeline, spawned as child process by Express
3. **Vite** — NOT running in prod. Only in dev mode (`run-dev.bat`)

Pipeline Manager (`pipeline-manager.ts`):
- Spawns Python with `child_process.spawn()`, pipes stdout/stderr with `[Gradio]` prefix
- Parses stdout for readiness: `"Running on local URL"` → state becomes `ready`
- Health check: polls `GET /gradio_api/info` every 10s, 3 consecutive failures → restart
- Auto-restart with exponential backoff (500ms → 15s, max 10 attempts)
- Graceful shutdown: `taskkill /pid PID /T /F` on Windows
- Opens browser automatically when pipeline is ready (first start only)
- Auto-finds free port if 3001 is busy (tries up to +10)

## Model Switching

Models switch **in-process** via Gradio's `POST /v1/init` API. NO process restart.

- Express `/api/generate/switch-model` → calls `POST http://localhost:8001/v1/init`
- Python handles: LM unload (`exit()` for vLLM to free CUDA graphs) → `gc.collect()` + `empty_cache()` → DiT reload (if changed) → LM reload
- `lm_backend` parameter passed through entire chain: Express → `/v1/init` → `llm_handler.initialize(backend=...)`
- `PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True` set in run.bat to reduce VRAM fragmentation
- Frontend polls `/api/generate/model-status` for state transitions: `unloading → loading → ready`
- LM model/backend dropdowns sync from server on every health poll (with `lmEditingRef` guard to not overwrite while user is editing)

**NEVER spawn a new Gradio process to switch models.** Always use `/v1/init`.

## Gradio Args (Express → Python)

Generation params are passed as **named parameters** via `@gradio/client` `predict()`:

```js
client.predict('/generation_wrapper', { captions: '...', lyrics: '...', auto_lrc: false, ... })
```

- Python `generation_wrapper()` has explicit named params (NOT `*args`)
- `gr.State` components (`is_format_caption`, batch indices) are hidden by Gradio client — do NOT include them in the Express object

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [timoncool/ACE-Step-Studio](https://github.com/timoncool/ACE-Step-Studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
