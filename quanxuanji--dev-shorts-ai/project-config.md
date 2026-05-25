---
trigger: always_on
description: DevShorts AI is an open-source voiceover script and FishSpeech-compatible audio generator for programmer-focused short videos. The priority is a visually strong local MVP that turns video/topic input into rewritten scripts, narration audio, subtitles, and a rendered `final.mp4`, while staying ready for future real model integrations.
---

# DevShorts AI Agent Guide

## Project Identity

DevShorts AI is an open-source voiceover script and FishSpeech-compatible audio generator for programmer-focused short videos. The priority is a visually strong local MVP that turns video/topic input into rewritten scripts, narration audio, subtitles, and a rendered `final.mp4`, while staying ready for future real model integrations.

The product should feel like an AI Studio / AI Runtime console, not a traditional CRUD admin panel.

## Current Architecture

- Frontend: `apps/web`, Next.js + TypeScript + TailwindCSS.
- Backend: `apps/api`, FastAPI + Python.
- Shared contracts: `packages/shared`.
- Docs: `docs/ARCHITECTURE.md`, `docs/API.md`, `docs/ROADMAP.md`.
- Task state: local in-memory store for now.
- Artifacts: `apps/api/artifacts/outputs/{taskId}/`.
- Runtime settings: `apps/api/artifacts/runtime-settings.json`, editable from `/settings`.

## Current Pipeline Modes

- `mock`: deterministic fallback pipeline for UI/API smoke tests.
- `semi_real`: local/video URL input, ffmpeg audio extraction/rendering, fallback ASR/LLM when providers are unavailable, real edge-tts optional, FishSpeech HTTP adapter, subtitles, final video output.

The semi-real target output is:

```text
apps/api/artifacts/outputs/{taskId}/final.mp4
```

## Provider Status

- LLM: `mock`, OpenAI-compatible, Ollama placeholders.
- ASR: `mock`, Whisper CLI, optional faster-whisper adapter.
- TTS: `mock`, `edge_tts`, `fishspeech` HTTP adapter.
- Digital human: not a core concept for the current product slice; keep any legacy adapter boundaries out of the main UX.
- Platform upload automation: not a core concept for the current product slice; do not present it as an active workflow.
- Render: ffmpeg-backed simple final video render.
- Settings: `/api/settings` can switch providers for new semi-real tasks.

Do not add heavyweight model runtimes directly to the repo. Prefer provider/adapters and graceful fallback logs.

## Local Startup

Install:

```powershell
npm install
python -m venv apps/api/.venv
apps/api/.venv/Scripts/Activate.ps1
pip install -r apps/api/requirements.txt
```

Run API:

```powershell
npm run dev:api
```

Run web:

```powershell
npm run dev:web
```

Open:

- Web: `http://localhost:3000`
- API docs: `http://localhost:8000/docs`

## Verification Commands

Before calling a change done, prefer:

```powershell
python -m compileall -f apps/api/app
npm run typecheck
npm run lint:web
npm run build:web
```

For media pipeline work, also create a `semi_real` task and verify:

- logs show each real/fallback step clearly.
- `voice.wav` is generated.
- `final.mp4` exists.
- `/artifacts/outputs/{taskId}/final.mp4` is accessible from the API.
- `/api/settings` can be read after Settings UI changes.
- Real ASR can be installed with `pip install -r apps/api/requirements-asr.txt`.

## UI Direction

Keep the UI high-impact and screen-record friendly:

- dark AI console style.
- purple/blue accents, but not cheap cyberpunk neon.
- glass panels, subtle glow, animated status, and deep dark spatial backgrounds.
- workflow timeline must clearly show pending/running/success/error.
- runtime console should feel alive.
- avoid plain enterprise dashboard styling.
- real `/studio` should feel like an AI Studio / Creator Pipeline, not a form page or a showcase poster.
- Wide desktop is the primary target. 3K/2K/1920 screens should show the left workflow, center workspace, right preview/assets/logs, and bottom timeline/status strip without feeling like a long marketing page.
- UI must be interactive, not just presentational: workflow steps should switch the workspace, scenes should be selectable, preview/export/log panels should respond to real task state.
- Showcase pages may use demo mode, but runtime `/studio` must distinguish real data, pending, waiting, and empty states clearly.

The app currently supports Chinese/English UI switching. Prefer Chinese copy for user-facing defaults unless a specific technical label is better in English.

## Hard-Won Runtime Rules

These rules came from real failures during the first generated video validation. Do not regress them.

### TTS and Voice Consistency

- Use segmented TTS for multi-scene videos: generate `scene-0.wav`, `scene-1.wav`, etc., then concatenate with silence.
- Do not return to a single long TTS blob and infer scene boundaries from punctuation.
- Every scene narration must contain its own rank prefix, for example `第一，...`, `第二，...`.
- Rank words such as `第二/第三/第四` must belong to the next scene, never the previous scene tail.
- Every scene in one video must use the same voice preset/reference. If FishSpeech is used, keep the same `reference_id` / reference audio / voice profile across all scene TTS calls.
- FishSpeech does not provide a reliable built-in voice library UI. The product should maintain its own voice profiles / fixed presets when users find a voice they like.

### Audio/Visual Sync


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quanxuanji/dev-shorts-ai](https://github.com/quanxuanji/dev-shorts-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-25 -->
