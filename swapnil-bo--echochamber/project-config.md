---
trigger: always_on
description: Handles: PDF file path, Wikipedia URL, any HTTP URL, raw pasted text.
---

# CLAUDE.md — EchoChamber: Auto-Podcast Generator
## Master Build Prompt for Claude Code

---

## 🎯 Project Overview

**EchoChamber** is a full-stack AI pipeline that transforms any URL, PDF, or Wikipedia page into a 5-minute, two-host audio podcast. Two AI personas (a skeptical host and an enthusiastic expert) debate the content, producing a structured script that is synthesized into audio using two distinct TTS voices, stitched together with intro/outro music, and served via a sleek React frontend with a real-time progress tracker.

This is a **portfolio project** targeting AI/ML Product Management roles. Code must be clean, well-commented, and structured so every architectural decision is explainable in an interview.

---

## 📁 Project Structure

```
echochamber/
├── backend/
│   ├── main.py                    # FastAPI app, CORS, startup hooks, health endpoint
│   ├── routers/
│   │   └── podcast.py             # /generate and /status/{job_id} endpoints
│   ├── pipeline/
│   │   ├── extractor.py           # Text extraction (URL, PDF, Wikipedia)
│   │   ├── chunker.py             # LangChain text chunking + summarization
│   │   ├── script_generator.py    # Gemini Flash — generates JSON script
│   │   ├── tts_engine.py          # edge-tts async TTS per line with retry
│   │   └── audio_mixer.py         # pydub stitching, normalization, fade
│   ├── utils/
│   │   ├── cache.py               # Script JSON cache + per-line TTS cache
│   │   └── job_store.py           # In-memory job status store
│   ├── static/
│   │   ├── intro.mp3              # Royalty-free intro music (pixabay.com)
│   │   └── outro.mp3              # Royalty-free outro music (pixabay.com)
│   ├── outputs/                   # Generated final podcast .mp3 files (gitignored)
│   ├── cache/
│   │   ├── scripts/               # Cached JSON scripts keyed by content hash (gitignored)
│   │   └── tts_lines/             # Cached per-line .mp3 files keyed by hash (gitignored)
│   ├── requirements.txt
│   ├── build.sh                   # Render build script — installs ffmpeg
│   └── .env
├── frontend/
│   ├── src/
│   │   ├── App.jsx
│   │   ├── components/
│   │   │   ├── InputForm.jsx          # URL/PDF/Wiki input with auto-detection
│   │   │   ├── ProgressTracker.jsx    # Glass-box live pipeline status
│   │   │   ├── AudioPlayer.jsx        # HTML5 audio player + onError handler
│   │   │   └── Transcript.jsx         # Script preview below audio player
│   │   └── index.css
│   ├── package.json
│   └── vite.config.js
├── .gitignore
└── CLAUDE.md
```

---

## 🛠️ Final Tech Stack

| Layer | Tool | Notes |
|---|---|---|
| Web Scraping | `requests` + `BeautifulSoup4` | |
| PDF Parsing | `PyMuPDF` (fitz) | |
| Wikipedia | `wikipedia-api` | |
| Text Chunking | `LangChain` text splitters + map_reduce | Hard cap at 1500 words |
| LLM | `Gemini 2.0 Flash` via `google-generativeai` | Free tier, 1500 req/day |
| TTS Voice A — Skeptic Host | `edge-tts` → `en-US-GuyNeural` | Async |
| TTS Voice B — Expert | `edge-tts` → `en-US-JennyNeural` | Async |
| TTS Retry | `tenacity` | 3 attempts, exponential backoff |
| Audio Stitching | `pydub` + `ffmpeg` | |
| Async CPU Offload | `asyncio.to_thread` | Prevents event loop blocking |
| Backend | `FastAPI` + `asyncio` | Async job pipeline |
| Job State | In-memory dict (`job_store.py`) | No Redis needed |
| Frontend | `React` + `Vite` + `TailwindCSS` | |
| Frontend Deploy | `Vercel` | |
| Backend Deploy | `Render` free tier (512MB RAM) | |

---

## ⚙️ Backend Implementation

---

### `main.py` — App Entry Point

This file handles four critical responsibilities:

**1. CORS Middleware (required for Vercel ↔ Render communication)**

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        os.getenv("ALLOWED_ORIGINS", "http://localhost:5173"),  # Vercel prod URL
        "http://localhost:5173",                                 # Local dev
    ],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

Without this, the browser will silently block all requests from the Vercel frontend to the Render backend due to CORS policy. This is non-negotiable.

**2. Startup Directory Initialization**

```python
@app.on_event("startup")
async def startup_event():
    # Create required directories if they don't exist.
    # These are gitignored and won't exist on a fresh clone or Render deploy.
    os.makedirs("outputs", exist_ok=True)
    os.makedirs("cache/scripts", exist_ok=True)
    os.makedirs("cache/tts_lines", exist_ok=True)
```

Without this, the first generation attempt will crash with `FileNotFoundError` on a fresh Git clone or Render deploy.

**3. Health Check Endpoint (Render cold start UX fix)**

```python
@app.get("/health")
async def health():
    return {"status": "ok"}
```

The React frontend pings this before starting a generation. If Render has been idle for 15+ minutes, the instance needs 30–50 seconds to boot. The frontend detects a slow or failed health check and shows a user-friendly "Backend waking up..." message instead of silently hanging.

**4. Static file serving**

```python
from fastapi.staticfiles import StaticFiles
app.mount("/audio", StaticFiles(directory="outputs"), name="audio")
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Swapnil-bo/EchoChamber](https://github.com/Swapnil-bo/EchoChamber) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
