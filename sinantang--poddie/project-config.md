---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Poddie is a **local, personal-use** Electron desktop app for editing podcasts by editing their transcript (Descript-style): import a recording (iPhone video, or audio-only m4a/mp3/wav/…) → transcribe with Whisper (word-level timestamps) → delete words/silences in the transcript or on the waveform → preview live → export the cut media with ffmpeg. Audio-only sources run the same pipeline minus video: `MediaInfo.hasVideo` gates the video export and caption burn-in. Single user, no cloud sync, no multi-platform packaging.

## Commands

```bash
npm run dev         # electron-vite dev (main process runs with --watch/hot-restart)
npm run build        # electron-vite build → out/
npm run typecheck    # tsc --noEmit
npm run lint         # eslint src tests
npm test             # vitest run, excludes the Whisper e2e test
npm run test:e2e     # PODDIE_E2E=1 vitest run tests/whisper-e2e.test.ts — hits the REAL OpenAI API, costs money
npm run dist:dir     # package unsigned universal .app into dist/mac-universal/ (test this first)
npm run dist         # package + build the .dmg into dist/
```

Packaging gotchas (hard-won; details in the findings/task_plan errors table): `build/afterPack.cjs` ad-hoc re-signs the bundle — without it Apple Silicon silently kills the app at launch. electron-builder's `files` is a deliberate **allowlist** (`out/**` + `package.json`); anything referenced at runtime via an electron-vite `?asset` import resolves to the app root (project root in dev, app.asar in prod) and must either be in that list or guarded dev-only (`!app.isPackaged`). When a packaged app "doesn't launch": run `dist/mac-universal/Poddie.app/Contents/MacOS/Poddie` directly in a terminal (stderr shows the real error), and remember a missing "Poddie Helper (Renderer)" process means no window was ever created even if main/GPU processes are alive. To inspect a packaged app's rendered DOM/console, launch it with `--remote-debugging-port=9222` and drive it over CDP (Node 22 has a global `WebSocket`). **Quit `npm run dev` before running `npm run dist:dir`/`dist`** — both write `out/`, and a build racing the live dev server can package a half-written `index.html` (which renders its linked CSS as raw text).

Run a single test file: `npx vitest run tests/edit.test.ts`. Run a single test by name: `npx vitest run -t "pattern"`.

**Never edit files under `src/` while a user has an export running** — the dev server's `--watch` hot-restarts the main process, orphaning any in-flight ffmpeg export and producing a corrupt (moov-atom-missing) output file. This has happened before (see task_plan.md errors table).

## Architecture

Three-process Electron layout: `src/main` (Node, ffmpeg/fs/OpenAI access), `src/preload` (typed bridge), `src/renderer` (React UI). The IPC contract — every channel name and the `PoddieApi` shape exposed to the renderer as `window.poddie` — is centralized in `src/shared/types.ts`; add new IPC there first, then implement the handler in `src/main/index.ts` and wire the preload.

**Non-destructive edit model (the core abstraction).** The source video is never touched until export. An edit is a flat sequence of `EditItem` (`src/shared/edit.ts`): each item is a `word` (from the transcript) or a `gap` (a silence ≥ 0.35s, treated as an equally deletable token — no special-casing silence vs. words). Every item has `removed: boolean`. `keptRanges()` is the **single derived artifact** — the complement of merged removed ranges over `[0, duration]` — and it alone drives preview seek-skipping, waveform cut-shading, the ffmpeg export graph, and caption timeline remapping. Anything that needs "what's left after edits" calls this function; nothing else computes ranges independently.

Undo/redo, cuts, in-place text edits (Phase 5.1), and silence auto-trim are all the same operation underneath: an `ItemChange` is a reversible field patch (`{index, prev: ItemPatch, next: ItemPatch}`) on one item. Because item *count* never changes, indices recorded in undo history stay valid forever — there's one `applyChanges()` code path for every kind of edit. **Invariant, enforced by tests:** display text is decorative — cuts/`keptRanges`/export derive only from `start`/`end`/`removed`, never from `text`, so a text edit or token merge must produce a byte-identical export.

**Persistence**: a `<video>.poddie.json` file next to the source video (`src/main/project.ts`), atomic write, holding the transcript and `EditState`. No database.

**Media pipeline** (`src/main/media.ts`, `ffmpeg.ts`): ffprobe for metadata; iPhone HEVC needs an H.264 proxy for Chromium preview (`ensurePreviewProxy`, hardware `h264_videotoolbox` → `libx264` fallback) — **exports always cut the original file, never the proxy**. Waveform peaks are precomputed in the main process (not decoded in the renderer) and cached, versioned so a density change invalidates stale cache entries.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SinanTang/poddie](https://github.com/SinanTang/poddie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
