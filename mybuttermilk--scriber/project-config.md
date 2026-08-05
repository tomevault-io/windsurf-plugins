---
trigger: always_on
description: Last verified: 2026-07-21
---

# Scriber Agent Guide

Last verified: 2026-07-21

This is the working guide for agents editing Scriber. Keep it current when the
implementation changes. Prefer code and tests over older prose when they
conflict, then update the docs in the same task.

## Active Documentation

The repository intentionally keeps only a small documentation set:

- `README.md`: user-facing overview, setup, configuration, and basic commands.
- `AGENTS.md`: this editing guide.
- `docs/ARCHITECTURE.md`: current system architecture and ownership boundaries.
- `docs/PERFORMANCE_AND_PACKAGING.md`: implemented performance work, Profile B
  ffmpeg, sidecar packaging, installer size, and remaining size/perf ideas.
- `docs/TESTING_AND_RELEASE.md`: test commands, smoke gates, installer builds,
  CI, signing, and updater status.
- `docs/ROADMAP_AND_KNOWN_ISSUES.md`: current open issues and prioritized next
  work.

Old implementation journals and superseded analysis docs were removed in the
2026-06-09 consolidation. Do not recreate fragmented one-off status files unless
the user explicitly asks for a temporary investigation note.

## Product Snapshot

- Scriber is an AI transcription app for live microphone dictation, bot-free
  meeting capture, YouTube transcription, file transcription, transcript
  management, summaries, and PDF/DOCX export.
- Primary desktop runtime: Tauri 2 shell, React frontend, Python backend sidecar.
- Backend default: `127.0.0.1:8765`, implemented with `aiohttp`, WebSocket
  events, SQLite, Pipecat pipeline code, and provider adapters.
- Frontend default in dev: `localhost:5000`, implemented with Vite 8, React 19,
  TypeScript, Tailwind v4, Wouter, and TanStack Query.
- Runtime is Windows-first. Linux/macOS support is mostly fallback/dev support.
- Legacy Python tray/UI code was removed. The Tauri shell owns desktop UI,
  tray/menu actions, global hotkeys, and the recording overlay.
- The pre-created recording-overlay WebView must register its native event
  listener before completing `native_overlay_renderer_ready`; that handshake
  returns the authoritative current snapshot so a hotkey fired during lazy
  renderer startup cannot leave a visible but transparent popup.

## Repository Map

Backend and runtime:

- `src/web_api.py`: main aiohttp controller, routes, WebSocket server, settings,
  jobs, transcript history, mic control, uploads, logs, support bundles.
- `src/pipeline.py`: STT pipeline orchestration, provider factory, analyzer
  cache, mic resolution, async/direct transcription.
- `src/core/provider_audio_formats.py` and `src/audio_prepare.py`: exact
  provider/route/model audio-format registry, ffprobe container+codec
  validation, pass-through-first batch selection, bounded ffmpeg preparation,
  and task-scoped cleanup of generated upload artifacts.
- `src/modulate_stt.py`: Modulate multilingual batch and streaming adapters.
  Both paths expose final transcript text only; they explicitly disable
  diarization, partials, emotion, accent, deepfake, and PII/PHI signals and
  discard provider utterance metadata at the boundary.
- `src/microphone.py`: live microphone capture boundary backed by the Rust
  WASAPI frame-pipe source, channel selection, RMS callback, stream lifecycle.
- `src/mic_prewarm.py`: Rust/WASAPI idle mic prewarm and rolling prebuffer.
- `src/device_monitor.py`: microphone hotplug monitor, native Windows endpoint
  callbacks, polling fallback, PortAudio refresh deferral.
- `src/audio_devices.py`: microphone normalization, compatibility filtering, and
  private PortAudio-to-native endpoint mapping with redacted endpoint hashes.
- `src/audio_file_input.py`, `src/youtube_download.py`, `src/runtime/media_tools.py`:
  ffmpeg/ffprobe resolution and media preparation. YouTube extraction uses the
  pinned yt-dlp/EJS/QuickJS-ng runtime and validates downloaded audio before
  provider upload. Frozen builds accept only the complete manifest-bound
  wrapper bundle under `tools/ffmpeg`; source runs may use the explicit
  `SCRIBER_QUICKJS_DEV_WRAPPER_PATH` override, but never an arbitrary `qjs`
  from `PATH`.
- `src/database.py`: SQLite WAL persistence, metadata loading, FTS5 search.
- `src/data/job_store.py`: persistent file/YouTube jobs.
- `src/data/latency_metrics_store.py`: hot-path metrics.
- `src/core/`: contracts, state machine, circuit breaker, logging, tracing.
- `src/runtime/audio_frame_pipe.py`: Python decoder/validator for the Rust
  audio frame-pipe protocol.
- `src/runtime/provider_http.py`: event-loop-owned reusable aiohttp provider
  transport, bounded DNS/connection pooling, and privacy-safe request timing.
- `src/native_overlay.py`: Python facade for the Tauri-owned recording overlay
  exposed through private shell IPC.
- `src/main.py`: compatibility notice for the removed Python desktop UI; use
  Tauri for desktop runs.

Frontend and shell:

- `Frontend/client/src/App.tsx`: routes; the five primary user tabs are eager,
  while Debug Console, transcript detail, and not-found surfaces remain lazy.
- `Frontend/client/src/pages/`: Live Mic, Meetings, YouTube, File, Settings,
  Debug Console, Transcript Detail.
- `Frontend/client/src/contexts/WebSocketContext.tsx`: shared WebSocket.
- `Frontend/client/src/lib/backend.ts`: backend URL and Tauri token bridge.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MyButtermilk/Scriber](https://github.com/MyButtermilk/Scriber) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
