---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Voice Studio by MSR — a fully-offline, local web UI for **multiple open-source TTS engines** (VibeVoice-1.5B, Kokoro-82M, Kitten TTS Mini, Chatterbox Multilingual V3, OmniVoice, VoxCPM2, Qwen3-TTS CustomVoice) plus Whisper speech-to-text. FastAPI backend serves the models; React + Vite frontend is a multi-segment podcast editor. Only one engine is loaded at a time to keep memory low.

## Commands

Primary entry point (cross-platform, from repo root):
```bash
python studio.py setup            # one-time: venv, deps, PyTorch/CUDA auto-detect, system-dep checks, model picker
python studio.py start            # run backend + frontend together (auto dev/prod)
python studio.py start --dev      # force dev (uvicorn :8880 + Vite :5173, hot reload)
python studio.py start --prod     # force prod (single server :8880 serving UI + API)
python studio.py models           # re-open the interactive model picker
```
`studio.py` is stdlib-only and bootstraps `backend/venv`; it forwards `start` flags (`--device`, `--port`, …) to `backend.cli`. The raw commands below are the underlying primitives.

Backend (run from repo root; package is `backend`):
```bash
python -m backend.cli --engine vibevoice --device cuda   # start server :8880; --device cpu|mps|auto
python -m backend.cli --help                             # all flags (--engine, --model, --kokoro-lang, --chatterbox-lang, --models-dir, --port)
cd backend && python -m pytest tests/                    # smoke tests (stubbed model, no weights needed, ~seconds)
cd backend && python -m pytest tests/test_smoke.py::<name>   # single test
```

Frontend (from `frontend/`):
```bash
npm run dev        # Vite :5173, proxies /api/* → :8880 (no CORS setup needed)
npm run typecheck  # tsc -b --noEmit
npm run build      # tsc -b && vite build
```

Two terminals: one backend, one frontend. First backend boot downloads model weights to `backend/models/` (VibeVoice ~5.4 GB). On Windows, install a CUDA PyTorch wheel *before* `pip install -r backend/requirements.txt` or CUDA silently falls back to CPU.

## Architecture

The backend is organized around a pluggable **engine abstraction**. Understanding it requires reading across `core/engines/`, `core/engine_manager.py`, and `services/synthesize.py`:

- **`core/engines/__init__.py`** — the `Engine` ABC. Each engine (`vibevoice_engine.py`, `kokoro_engine.py`, `chatterbox_engine.py`) is a self-contained model+processor implementing `load/unload/is_loaded/synthesize/...`. Engines are constructed at startup but loaded **lazily** on first use. `EngineSynthRequest`/`EngineResult` are the engine-agnostic I/O dataclasses; engines ignore fields they don't understand (e.g. Kokoro ignores `reference_audio`; only Chatterbox uses `cfg_weight`/`exaggeration`/`language_id`).
- **`core/engine_manager.py`** — `EngineManager` owns the engine registry and tracks the single active engine. Switching engines unloads the previous one. The active choice is persisted to `backend/.last_engine` and restored on restart. **Register new engines here** (the dict order drives the UI selector).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [msrbuilds/voice-studio](https://github.com/msrbuilds/voice-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
