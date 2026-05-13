---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## About

Argo turns Playwright demo scripts into polished product demo videos with AI voiceover. Users write a Playwright test, add a scenes manifest (`.scenes.json`), and run `argo pipeline` to get an MP4 with screen recording, overlays, and narration — all locally, no cloud services required.

## Build & Test

- `npm run build` — TypeScript compilation (strict mode, ESM, target ES2022)
- `npm test` — runs vitest (all unit + integration tests)
- `npx vitest run tests/path/to/test.ts` — run a single test file
- E2E tests require Playwright browsers: `npx playwright install chromium`
- No separate lint command currently configured
- Kokoro TTS defaults: model `onnx-community/Kokoro-82M-v1.0-ONNX`, dtype `q8`
- Clear TTS cache if voiceover text changes: `rm -rf .argo/<demo>/clips`

## Publishing

- Package: `@argo-video/cli` (npm org: `@argo-video`)
- Publishing is automated via GitHub Actions OIDC trusted publishing (no NPM_TOKEN needed)
- To release: bump version in package.json, tag, create GitHub release → workflow handles the rest
- IMPORTANT: The `exports` map in package.json must include a `"default"` condition alongside `"import"` — without it, consuming projects that lack `"type": "module"` fail with `No "exports" main defined`

## Git Conventions

- `demos/` directory is no longer gitignored — demo source files are tracked normally
- `videos/` directory is also gitignored — use `git add -f videos/<file>` for tracked video artifacts
- GPG signing may fail in CLI environments — use `git -c commit.gpgsign=false commit` if needed

## Architecture

The system is a 4-step pipeline: **TTS → Record → Align → Export**

- **TTS** (`src/tts/`): Generates voice clips from `.scenes.json` manifests. Pluggable engine system with 7 built-in adapters in `src/tts/engines/`: Kokoro (default), OpenAI, ElevenLabs, Gemini, Sarvam, mlx-audio, Transformers (generic HuggingFace pipeline). Selected via `engines.*` factory functions in config. Cloud engines lazy-load their SDKs. All audio normalized to mono Float32 24kHz WAV via `convertToWav()` (ffmpeg). Clips are content-addressed cached (SHA256 of scene+text+voice+speed+lang) in `.argo/<demo>/clips/`. Cloud engine API keys are validated lazily at `generate()` time, not constructor — so non-TTS commands like `argo validate` work without keys. Kokoro's ONNX runtime is not safe for concurrent `generate()` calls — clips generate sequentially despite shared init promise.
- **Record** (`src/record.ts`): Runs Playwright demo script under `@playwright/test`. Recording itself is driven by `narration.startRecording(page)` inside the demo (`page.screencast.start` from Playwright 1.59), not by Playwright's auto `recordVideo`. The generated config sets `video: 'off'` and the screencast lands at `.argo/<demo>/video.webm` via `ARGO_SCREENCAST_PATH`. Critical: `trace: 'on'` and `screencast.start()` cannot coexist — both engines pin the page render at trace's snapshot resolution (800×450 on webkit, intermittent white frames on chromium), so the generated config keeps trace off. The `screencast.start()` call passes `onFrame` when either `ARGO_LIVE_FRAME_PATH` (preview live thumbnail) or `ARGO_THUMBS_DIR` (per-scene scrubber JPEGs) is set; `narration.mark()` flips a `_pendingThumbScene` flag that the next `onFrame` consumes to write `<thumbsDir>/<scene>.jpg`. Optional `video.showActions: true | { position, fontSize, duration }` calls `page.screencast.showActions()` immediately after `start()` to auto-annotate every Playwright interaction (clicks/fills) in the recording.
- **Align** (`src/tts/align.ts`): Places audio clips at scene timestamps from timing data. Prevents overlap with 100ms gaps. Mixes into single WAV (Float32, 24kHz).
- **Export** (`src/export.ts`): Merges video + aligned audio via ffmpeg into final MP4. Supports optional MP4 thumbnail embedding via `export.thumbnailPath` config (ffmpeg attached_pic stream). CRITICAL: `-shortest` must be skipped when thumbnail is present — PNG has 0 duration and truncates the entire output. Embeds chapter markers from scene placements via ffmpeg metadata. Input indices are dynamic based on presence of chapters/thumbnail. Silent mode: when no `narration-aligned.wav` exists, exports video-only (no audio input, no `-c:a`, no `-shortest`). Shows a progress bar during encoding when total duration is known (uses ffmpeg's `-progress pipe:1`). Supports multi-format export: `1:1` (square blur-fill), `9:16` (vertical blur-fill), and `gif` (two-pass palette-optimized animated GIF). Audio loudnorm: `export.audio.loudnorm` applies EBU R128 (-16 LUFS) via `-af loudnorm` or inside `filter_complex` when transitions are active. Viewport-native variants: `export.variants` re-records at different viewports (TTS shared, record+export per variant).
### Export Quality (`src/export.ts`, `src/gpu-encoder.ts`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shreyaskarnik/argo](https://github.com/shreyaskarnik/argo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
