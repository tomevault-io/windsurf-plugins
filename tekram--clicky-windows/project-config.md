---
trigger: always_on
description: Notes for Claude Code (and future contributors) working in this repo.
---

# CLAUDE.md

Notes for Claude Code (and future contributors) working in this repo.
High-level architecture first, then the tricky bits that are easy to get
wrong.

## What this is

Clicky Windows is an Electron + TypeScript port of
[farzaa/clicky](https://github.com/farzaa/clicky). It's a tray-resident AI
screen companion: the user speaks or types, Clicky captures a screenshot,
sends it to a vision LLM, and **points at UI elements with an animated
cursor overlay** based on POINT tags the model emits inline in its reply.

See `README.md` for user-facing docs and `docs/` for feature docs.

## Architecture

```
src/
├── main/                  # Electron main process (Node)
│   ├── index.ts           # Window creation, IPC wiring, app bootstrap
│   ├── companion.ts       # Central orchestrator:
│   │                      #   transcript → screen capture → AI query →
│   │                      #   POINT tag pipeline → overlay → TTS
│   ├── screenshot.ts      # desktopCapturer wrapper + crop helper
│   ├── audio.ts           # PCM coming from renderer → transcription provider
│   ├── hotkey.ts          # Global push-to-talk hotkey
│   ├── tray.ts            # System tray icon/menu
│   └── settings.ts        # Persistent settings store
├── services/              # External service clients (main process only)
│   ├── claude.ts          # Anthropic — pass-1 query + pass-2 refinePoint
│   ├── openai-chat.ts     # OpenAI chat completion (vision)
│   ├── openrouter-chat.ts # OpenRouter (routes to various models)
│   ├── transcription/     # AssemblyAI / OpenAI Whisper / whisper.cpp local
│   └── tts/               # ElevenLabs / OpenAI TTS / Windows SAPI
├── preload/               # contextBridge between main and renderer
└── renderer/              # HTML/CSS/JS for each window
    ├── chat/              # Chat window (text + push-to-talk mic)
    ├── overlay/           # Transparent fullscreen click-through cursor
    └── settings/          # Settings panel
```

### End-to-end query flow (what happens when the user asks something)

1. `renderer/chat` sends either a typed question via `chat:query` IPC, or
   after push-to-talk captures and decodes the mic blob to 16-bit PCM and
   sends it via `audio:recording-complete`.
2. `main/audio.ts` routes PCM to the configured transcription provider
   (`whisper-local` locally via whisper.cpp, or `openai` via the Whisper
   API). The transcript is passed to `CompanionManager.processQuery`.
3. `main/companion.ts` captures all screens (`ScreenCapture.captureAllScreens`),
   picks the current `AIProvider` (Claude / OpenAI / OpenRouter) and calls
   `query()` with the transcript, screenshots, cursor position, and trimmed
   conversation history.
4. The LLM responds with free-form text containing inline POINT tags in the
   format `[POINT:x,y:label:screenN]`.
5. `companion.ts` parses the raw tags, runs a **second-pass refinement**
   (see below), scales the resulting image-space coordinates to display
   pixels, and pushes them to the overlay via `overlay:point` IPC.
6. In parallel, the response text (with POINT tags stripped) is sent to the
   TTS provider and the raw text is returned to the chat window.

## Pointing pipeline — three coordinate spaces

**This is the easiest part of the codebase to break.** There are three
coordinate spaces in play and every transformation must be explicit:

1. **imageDimensions** — pixel space of the downsampled JPEG sent to the
   model in pass 1. Capped at `MAX_DIMENSION = 1568` because going higher
   triggers Anthropic's API-side downscaling, which silently invalidates
   model-emitted coordinates.
2. **native crop pixels** — pixel space of a high-DPI crop made from the
   original full-resolution `_source` NativeImage and sent to the model in
   pass 2 (refinement). This has a different ratio than imageDimensions.
3. **display pixels** — CSS coordinates for the overlay window, which is
   sized to `display.bounds.{width,height}`.

### Pass 1 → pass 2 refinement

A single LLM call gives roughly-right coordinates (±30 px on a 1568-wide
image is typical). On dense UI with adjacent similar elements (like/dislike,
tabs, toolbar buttons) that's not enough. The second pass:

1. For each POINT tag from pass 1, `cropScreenshotRegion` crops a ~300 px
   window (imageDimensions space) around the estimate. Internally it crops
   from the **full-resolution `_source` NativeImage**, not from the
   downsampled JPEG, so the model sees a genuinely higher-DPI patch.
2. `ClaudeService.refinePoint` sends this crop with a minimal, strict
   system prompt ("return only `x,y`, ignore visually similar neighbors")
   and `max_tokens: 32`.
3. Refined coords come back in native-crop pixels and must be translated:
   `imageX = origin.x + refined.x / pxPerImageDim`, then scaled to display
   pixels via `bounds.width / imageDimensions.width`.
4. All refinement calls run in `Promise.all`, so latency ≈ one extra API
   round-trip regardless of how many POINT tags the first pass emitted.

### Coordinate-space footguns

- **Never** send a pass-1 image larger than `MAX_DIMENSION`. Anthropic will
  downsample it server-side and the coords you get back will be in some
  third space you can't predict.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tekram/clicky-windows](https://github.com/tekram/clicky-windows) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
