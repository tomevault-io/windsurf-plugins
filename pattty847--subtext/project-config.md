---
trigger: always_on
description: This file is the fast context guide for coding agents and contributors working on Subtext.
---

# AGENTS.md - Subtext Contributor Guide

This file is the fast context guide for coding agents and contributors working on Subtext.

## Project Identity

Subtext is one local-first project with two companion modes:

- **Private web service**: iPhone/browser workflow for private URL download, URL transcription, and local file transcription over Tailscale
- **Desktop app**: full PySide workflow for local media processing, transcript review, Ollama analysis, and exports

Keep these modes coherent, but do not blur their boundaries in code or docs.

## Project Goals

- Keep setup simple for non-technical users.
- Prefer local-first processing and private-network access.
- Preserve UI responsiveness during heavy desktop work.
- Keep the private web service fast and dependable for repeat phone use.
- Favor clear status messages, safe fallbacks, and memory-stable defaults.

## Current Product Boundaries

### Private Web Service

- Runs via `run_web.py`
- Binds to `127.0.0.1:8000` by default
- Intended to be exposed privately through Tailscale Serve
- Supports:
  - URL transcription
  - local audio/video file transcription
  - URL video download
- Does **not** include the Desktop app’s transcript editing, Ollama analysis, or export workflow

### Desktop App

- Runs via `run.py`
- Uses PySide6 tabs and QThread workers
- Supports:
  - queued local + URL workflows
  - transcript review/editing
  - Ollama analysis
  - export formats

Do not describe or implement the web service as if it already has full desktop AI analysis features unless that is an explicit new feature request.

## Stack

- Python 3.11+
- PySide6 for desktop UI
- FastAPI for the private web service
- AsyncIO + QThread workers
- yt-dlp for media download/captions
- Whisper / optional faster-whisper for transcription
- Ollama for local LLM analysis
- `uv` for dependency management

## Code Boundaries

- `src/ui/`: Desktop Qt UI only
- `src/ui/workers/`: QThread wrappers for desktop async calls
- `src/web/`: private FastAPI service + static web UI
- `src/core/`: business logic and integrations; no direct UI updates
- `src/config/`: shared config and path constants
- `scripts/`: install/start/maintenance helpers
- `docs/`: human-facing technical/product docs

Do not place heavy business logic directly in Qt tab classes or browser-facing JS.

## Key Runtime Rules

1. Desktop UI updates must happen in the UI thread.
2. Long desktop tasks must run in workers.
3. Workers should communicate with signals only.
4. Async I/O belongs in core or web service modules, not Qt widgets.
5. Always use `ProjectPaths` from `src/config/paths.py`.
6. Keep the private web service localhost-only unless the user explicitly requests a different security model.
7. Prefer Tailscale/private-network patterns over public exposure.

## Current Processing Behavior

- Input can mix URLs and local files in the desktop app.
- YouTube URLs can use captions-first fast path.
- If captions fail, are unavailable, or rate-limit, fallback is Whisper.
- Batch work is sequential by design for memory stability.
- Desktop processing unloads Whisper resources after runs.
- Private web service keeps the transcription model warm across requests.
- Ollama calls use low-memory behavior and should remain desktop-focused unless intentionally expanded.

## Private Web Service Expectations

- Entry point: `run_web.py`
- Main service: `src/web/server.py`
- Static client: `src/web/static/`
- Auth model:
  - shared secret via `SUBTEXT_SERVER_KEY`
  - optional IP allow rules
  - Tailscale is the intended access path
- Current web UX is mobile-first and optimized for Safari on iPhone.
- Browser download flows should prefer attachment-style responses over fragile JS-only download tricks.

When changing the web service:

- Preserve localhost binding by default.
- Do not reintroduce old LAN/`0.0.0.0`/same-Wi-Fi assumptions in code or docs.
- Keep the Safari/iPhone path smooth.
- Treat download and transcription as first-class web tasks.

## Desktop App Expectations

- Main app shell: `src/ui/main_window.py`
- Media workflow: `src/ui/download_tab.py`
- AI workflow: `src/ui/analysis_tab.py`
- Results/export: `src/ui/results_tab.py`

When changing desktop behavior:

- Keep heavy work out of widgets.
- Preserve signal-driven state transitions.
- Keep transcript-to-analysis flow intact.

## Common Feature Entry Points

### New desktop download/transcript option

- `src/ui/widgets/multi_select_dropdown.py`
- `src/ui/download_tab.py`
- `src/ui/workers/download_worker.py`
- `src/core/processor.py`

### New desktop AI analysis feature

- `src/core/analyzer.py`
- `src/ui/workers/analysis_worker.py`
- `src/ui/analysis_tab.py`
- `src/ui/results_tab.py`

### New private web service feature

- `src/web/server.py`
- `src/web/static/index.html`
- `src/web/static/app.js`
- `src/web/static/style.css`
- `src/core/downloader.py` / `src/core/transcriber.py` as needed

## Conventions

- Type hints required on public methods/functions.
- Keep methods focused and reasonably short.
- Prefer explicit names over abbreviations.
- Keep docstrings concise and practical.
- Use `pathlib.Path`, not raw path strings.
- Avoid broad `except:`; catch specific exceptions where possible.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pattty847/Subtext](https://github.com/pattty847/Subtext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
