---
trigger: always_on
description: Orly (from OveRLaY) is a real-time AI agent that lives on your desk. A camera sees the table surface. A Gemini Live API session hears you and sees your world through a single streaming session. The agent speaks back and calls tools to physically project images, diagrams, stories, music, and more onto the table via a mini projector. No screen. No headset. Output lands on the physical desk.
---

# AGENTS.md

## What This Is

Orly (from OveRLaY) is a real-time AI agent that lives on your desk. A camera sees the table surface. A Gemini Live API session hears you and sees your world through a single streaming session. The agent speaks back and calls tools to physically project images, diagrams, stories, music, and more onto the table via a mini projector. No screen. No headset. Output lands on the physical desk.

It's not just a tutoring system — it's a seamless blend of digital and material. Help with homework, create illustrated stories together, explore the world, generate music, or just have a conversation with an AI that can see and act in your physical space.

Being built for the [Gemini Live Agent Challenge](https://geminiliveagentchallenge.devpost.com/) hackathon (Live Agents category). Full plan in `PROJECT_PLAN.md`.

## Architecture

Two components connected by WebSocket:

**Cloud Run backend** (`backend/`) — FastAPI + raw google-genai SDK. Uses `client.aio.live.connect()` to establish a bidirectional Gemini Live session. Audio and video are sent as separate concurrent streams (no FIFO queue). Tool calls are handled directly in the receive loop. Session resumption and context window compression are configured natively. Deployed on GCP.

**Local edge client** (`client/`) — Python asyncio app. Captures camera via IP Webcam, runs ArUco marker detection + homography (OpenCV), captures mic audio, sends rectified frames + PCM audio to backend over WebSocket. Receives audio responses (plays through speakers) and tool results (renders overlays via matplotlib, maps to projector coordinates via homography, displays on projector or screen).

**Simulation harness** (`simulation/`) — Synthetic audio/video pipeline for testing without hardware. Generates fake PCM audio (silence, sine waves, TTS), fake camera frames (JPEG), connects to the real backend, and measures end-to-end latency. Pre-defined scenarios for smoke testing and benchmarking.

The coordinate system is the key insight: Gemini's 0–1000 bounding boxes map directly to normalised table coordinates, which map to projector pixels through a calibrated homography. See `PROJECT_PLAN.md` §3.2.

## Stack

- Python 3.12+
- `google-genai` — Raw Gemini SDK (Live API for bidirectional audio/video streaming)
- `fastapi` + `uvicorn` — WebSocket server
- `opencv-contrib-python` — ArUco detection, homography, image warping
- `numpy` — matrix math
- `matplotlib` — overlay rendering (graphs, diagrams)
- `pyaudio` — mic capture + speaker playback
- `websockets` — edge client WS connection

## Key Technical Decisions

- **Raw GenAI SDK, not ADK.** ADK's `LiveRequestQueue` serializes all input (audio, video, text) into a single FIFO queue. Audio gets stuck behind video frames, adding ~5s speech-to-transcription latency. The raw SDK natively supports separate `audio=`/`video=` streams via `session.send_realtime_input()`. The backend went from 437 lines of workarounds to ~250 lines of clean code.
- **Tools as plain Python functions.** `function_to_declaration()` in `agent.py` auto-generates JSON tool schemas from Python function signatures + docstrings. Same convenience as ADK, no dependency.
- **Vertex AI on Cloud Run, not Google AI API key.** Service account auth, no key management, counts as a Google Cloud service for hackathon requirements. Google AI API key for local dev.
- **Black background for all overlays.** Projectors add light — black is transparent. All rendering must use dark backgrounds with bright content.
- **Homography caching.** Camera homography is cached on last-good-detection so brief marker occlusion doesn't break the system.
- **Screen overlay as fallback.** If no projector is connected, overlays composite onto the rectified camera view in a laptop window. Same code path, different output target.

## Coding Rules

1. **Red/green TDD.** Write a failing test first. Make it pass. Then refactor. No untested code lands in main.
2. **Commit at logical intervals.** One commit per coherent change — a new function, a bug fix, a refactor. Not "end of day" dumps. Write descriptive commit messages.
3. **Review and refactor regularly.** After each PoC milestone, review the code. Extract shared utilities. Remove dead code. Improve names. Don't let tech debt compound.

## File Layout

```
orly/
├── CLAUDE.md              ← you are here
├── PROJECT_PLAN.md        ← full plan, architecture, hackathon strategy
├── backend/               ← Cloud Run service (FastAPI + raw GenAI SDK)
│   ├── main.py            ← WebSocket endpoint + Gemini Live session
│   ├── agent.py           ← System prompt + tool schema generation
│   └── tools.py           ← project_overlay, refresh_view, show_scene, run_program, stop_program, list_programs, get_overlay_state, generate_code, generate_video, play_video, stop_video, play_music, stop_music, pause_music, resume_music, replay_music, get_session_manifest

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Pugio/Orly](https://github.com/Pugio/Orly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
