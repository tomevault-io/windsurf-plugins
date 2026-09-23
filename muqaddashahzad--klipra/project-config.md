---
trigger: always_on
description: **Read this file first. If you need more depth, then read `CLAUDE.md`.**
---

# AGENTS.md — Klipra Quick Brief for AI Agents

**Read this file first. If you need more depth, then read `CLAUDE.md`.**

This is a 60-second briefing so any AI agent (Claude, Cursor, ChatGPT, Hermes, etc.) can start working on this codebase without reading the whole repo.

---

## What Klipra is

A free, self-hosted AI viral-video studio. Drop in a long video → get short vertical clips, subtitled, hooked, reframed, ready for TikTok / Reels / Shorts.

- **Owner & user:** Muqaddas Shahzad (muqaddas@ilmeaalim.com) — non-technical, runs the product side of ilmeaalim.com.
- **Local machine:** M1 Max MacBook Pro, 64 GB RAM, macOS, Docker Desktop.
- **Path on her Mac:** `/Volumes/Data/AntiGravity/Klipra`
- **GitHub (public, MIT):** [https://github.com/muqaddashahzad/Klipra](https://github.com/muqaddashahzad/Klipra)
- **Production URL:** [klipra.ilmeaalim.com](https://klipra.ilmeaalim.com) (Hetzner VPS 46.62.194.70, Plesk, Cloudflare DNS)
- **Forked from:** [OpenShorts](https://github.com/mutonby/openshorts) (MIT) — Klipra has diverged substantially.
- **Current version status:** ~250 task completions on top of the fork; June 2026 onwards is the public-release era.

---

## Four products inside Klipra

1. **Generate Viral Clips** — full pipeline, drop video → 4-8 shorts. `main.py` + `app.py`.
2. **Smart Clipper** — multimodal VLM picker (Fast/Pro modes). `multimodal_picker.py` + `smart_clipper_pro.py`.
3. **Standalone Subtitle** — burn subtitles on any video, three-phase flow. `subtitles.py`.
4. **Standalone Voice Dubbing** — ElevenLabs dub in 30+ languages, optional subtitle burn. `translate.py`.

A fifth tab — **AI Avatar** — exists as a UI placeholder for talking-head generation. Backend not wired up yet (LongCat-Video-Avatar requires CUDA, won't run on M1).

---

## Architecture (Docker Compose)

| Service | Container | Port | Purpose |
|---|---|---|---|
| `backend` | klipra-backend | 8000 | FastAPI + Uvicorn (`app.py`, ~520 KB) + all video processing |
| `frontend` | klipra-frontend | 5175 → 5173 | React 18 + Vite 4 + Tailwind |
| `renderer` | klipra-renderer | 4000 | Optional Remotion renderer |

Optional macOS-host sidecar: `whisper_sidecar/` — runs Whisper on Apple Silicon Metal GPU at 10-15× CPU speed.

**Bind mounts (data survives `docker compose down`):**
- `.:/app` — repo mounted live (Python edits no-rebuild)
- `./output`, `./uploads` — past projects + source videos
- `./data` — accounts SQLite db (**never push to GitHub**)
- `~/Desktop/Movies → /app/local_media:ro` plus expanded mounts to ~/Desktop, ~/Downloads, ~/Documents, ~/Movies

---

## Tech stack at a glance

- **Backend (Python 3.11):** FastAPI · Uvicorn · faster-whisper · mediapipe · ultralytics (YOLOv8) · yt-dlp · ffmpeg-python · google-genai · ollama-py · Pillow · pysubs2 · libass · SQLAlchemy/SQLite
- **Frontend:** React 18 · Vite 4 · Tailwind 3.4 · lucide-react
- **LLM providers wired up:** Gemini · OpenAI · Anthropic · OpenRouter · Groq · MiniMax · Ollama. xAI/Grok not yet (key generation deferred).
- **External services:** Google Gemini · ElevenLabs · Ollama · Upload-Post · AWS S3 · Hetzner+Plesk · Cloudflare
- **Free-first philosophy:** Ollama (Qwen 2.5 14B/32B) is the default. Gemini free tier is the secondary. Paid options exist but are never the default.

---

## How to run it

```bash
git clone https://github.com/muqaddashahzad/Klipra.git
cd Klipra
cp .env.example .env       # optional — all keys optional if using Ollama
docker compose up -d
# App at http://localhost:5175 ; API docs at http://localhost:8000/docs
```

Frontend hot-reloads on save (Vite HMR). Python files are live-mounted; sometimes need `docker compose restart backend` to kick. `requirements.txt` / `Dockerfile` / `package.json` changes need `docker compose up -d --build`.

Mac one-click helpers (double-click in Finder): `Restart-Backend.command`, `Force-Restart-Backend.command`, `Restart-All.command`, `Apply-File-Folder-Access.command`, `Install-Ollama-Model.command`, `Publish-To-GitHub.command`.

---

## How to push changes to GitHub

```
cd /Volumes/Data/AntiGravity/Klipra
git add -A
git commit -m "feat: <short summary>"
git pull --rebase origin main
git push origin main
```

Or double-click `Publish-To-GitHub.command`. Credentials are cached in macOS Keychain.

**Sandboxed AI agents** (Claude in Cowork, etc.) cannot push directly. They either:
- Ask Muqaddas to run the commands above, OR
- Use the **Hermes Agent** as a relay (see CLAUDE.md → "Hermes-as-hands workflow")

---

## Critical gotchas

- **`app.py` is ~520 KB.** Always use Grep, never full Read.
- **Past projects use `output/{job_id}/metadata.json`** as source of truth. Rehydrator in `app.py` reconstructs in-memory state from disk.
- **Container names are hardcoded** (`klipra-backend`, etc.). Only one stack per host.
- **Daily Gemini quota** = 20 RPD on free tier; main.py exits with code 2 on PerDayPerProject. Resume picks up where it stopped.
- **Switch-AI on resume:** `/api/retry/{job_id}` accepts `X-LLM-Provider`, `X-LLM-Model`, `X-LLM-Key`, `X-LLM-Base-URL` headers. Frontend has a "🔄 Switch AI & resume" button next to the regular Resume on failed jobs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [muqaddashahzad/Klipra](https://github.com/muqaddashahzad/Klipra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
