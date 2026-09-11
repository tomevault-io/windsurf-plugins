---
trigger: always_on
description: This file orients an AI assistant (Claude, Cursor, Codex, Copilot, …) that has just been pointed at the `movi-player` repo or has it installed as a dependency. It is **not** end-user documentation — see [README.md](./README.md) and [https://moviplayer.com](https://moviplayer.com) for that. The intent here is to make the assistant useful for debugging, extending, and integrating the player without re-discovering the architecture every conversation.
---

# AGENTS.md — `movi-player` for AI coding assistants

This file orients an AI assistant (Claude, Cursor, Codex, Copilot, …) that has just been pointed at the `movi-player` repo or has it installed as a dependency. It is **not** end-user documentation — see [README.md](./README.md) and [https://moviplayer.com](https://moviplayer.com) for that. The intent here is to make the assistant useful for debugging, extending, and integrating the player without re-discovering the architecture every conversation.

If you are an AI assistant: read this top to bottom once, then refer back by section when you take an action that touches the player.

---

## 1. What this is

`movi-player` is a browser video player that plays formats the native `<video>` element can't: MKV, HEVC, AV1, 4K HDR, multi-audio, embedded subtitles, encrypted streams. It does this with:

- **FFmpeg (libavformat) compiled to WebAssembly** for demuxing arbitrary container formats.
- **WebCodecs API** (Chrome, Safari 18+, Firefox 130+) for hardware-accelerated decoding when available, software dav1d/de265 in WASM as fallback.
- **Canvas/WebGL2** for presentation (the spec for `<canvas>` lets you render HDR via `display-p3` color space; native `<video>` can't expose that on most browsers).

Shipped as a web component (`<movi-player>`) plus three lower-level entry points (`/player`, `/demuxer`) for programmatic use. License: Apache-2.0.

The same engine ships in three wrappers built on the web component: an Electron **desktop app** (`desktop/`, Win/Mac/Linux — local files, URLs, native always-on-top PiP, playlist), a **Chrome extension** (`chrome-extension/`), and a **VS Code extension** (`vscode-extension/`).

---

## 2. Package entry points

Set in `package.json`. Three bundle tiers, picked by import path:

| Import | Bundle size | What you get |
|--------|-------------|--------------|
| `movi-player` or `movi-player/element` | ~410 KB | Full web component including all controls, gestures, ambient mode, subtitle rendering, chapters, etc. |
| `movi-player/player` | ~180 KB | `MoviPlayer` class — programmatic playback orchestration without UI. Bring your own canvas + controls. |
| `movi-player/demuxer` | ~45 KB | `Demuxer`, source adapters, decoders. For metadata extraction, thumbnail generation, manual pipelines. |

WASM blob (`dist/wasm/*.wasm`) is loaded lazily on first use and shared across instances (~2 MB compressed).

The three entry points re-export progressively more of the codebase. `element` re-exports everything in `player`, which re-exports everything in `demuxer`. So `import { Demuxer } from 'movi-player'` works too — it's just bigger.

---

## 3. Architecture map

Reading order if you've never touched this code:

```
src/
├── element.ts                 — entry, re-exports MoviElement
├── player.ts                  — entry, re-exports MoviPlayer
├── demuxer.ts                 — entry, re-exports Demuxer
├── index.ts                   — top-level entry (re-exports element.ts)
├── types.ts                   — public TypeScript types (start here)
│
├── render/
│   ├── MoviElement.ts         (~15k lines) Web component. UI, attributes,
│   │                          events, keyboard, gestures, menus, ambient
│   │                          mode, subtitles overlay, theming. Wraps a
│   │                          MoviPlayer instance.
│   ├── CanvasRenderer.ts      WebGL2 video presentation. HDR via display-p3
│   │                          on Chromium; PQ tone-map shader elsewhere.
│   │                          Owns the frame queue and A/V sync.
│   ├── AudioRenderer.ts       Web Audio scheduling, Signalsmith stretcher
│   │                          for non-1x rates, stable-audio compressor.
│   └── HLSPlayerWrapper.ts    Delegates to hls.js for HLS sources, with a
│                              compatible API surface.
│
├── core/
│   ├── MoviPlayer.ts          (~4500 lines) The orchestrator: load, play,
│   │                          pause, seek, demux loop, decoder wiring,
│   │                          backpressure, A/V sync, state machine.
│   ├── PlayerState.ts         State machine: idle → loading → ready →
│   │                          playing ⇄ paused ⇄ seeking ⇄ buffering →
│   │                          ended | error.
│   ├── Clock.ts               Media-time clock with playbackRate support.
│   ├── TrackManager.ts        Active video/audio/subtitle track selection.
│   └── PlaybackController.ts  High-level play/pause coordination.
│
├── demux/Demuxer.ts           Thin wrapper over WasmBindings.demux.
│
├── decode/
│   ├── VideoDecoder.ts        Hardware-first WebCodecs decoder with
│   │                          extensive fallback chain (color-space strip,
│   │                          HEVC Rext profile retries, software dav1d).
│   ├── AudioDecoder.ts        WebCodecs audio. Opus is force-software

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nyumat/NyumatFlix](https://github.com/Nyumat/NyumatFlix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
