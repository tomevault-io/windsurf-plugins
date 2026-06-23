---
trigger: always_on
description: Fully-local screen + camera recorder web app (Chrome-only APIs). Everything — capture, compositing, encoding, muxing, trim/convert/audio-enhance — runs in the browser. Nothing leaves the machine. MIT, owned by Amit Rawat (@sahajamit), part of the Agentic Engineer brand.
---

# framecast

Fully-local screen + camera recorder web app (Chrome-only APIs). Everything — capture, compositing, encoding, muxing, trim/convert/audio-enhance — runs in the browser. Nothing leaves the machine. MIT, owned by Amit Rawat (@sahajamit), part of the Agentic Engineer brand.

## Commands

| Task | Command |
|------|---------|
| Dev server | `npm run dev` (open in Chrome/Edge 122+) |
| Unit tests (vitest) | `npm test` |
| E2E (Playwright + real Chrome) | `npm run e2e` |
| Typecheck | `npx tsc -b` |
| Lint / format | `npm run lint` / `npm run format` |
| Build (includes typecheck) | `npm run build` |

CI (`.github/workflows/ci.yml`) runs lint + build + unit + e2e. `netlify.yml` deploys `dist/` to **https://framecast.amitrawat.dev** (Netlify site `framecast`, headers in `netlify.toml`) on every push to main. The old github.io URL serves a static redirect.

## Architecture (the 60-second version)

```
getDisplayMedia / getUserMedia
  → MediaStreamTrackProcessor readables (transferred to worker)
  → compositor.worker.ts: OffscreenCanvas draw (frame-driven + 1 Hz heartbeat)
  → MediaStreamTrackGenerator
  → mediabunny MediaStreamVideo/AudioTrackSource (WebCodecs H.264 + AAC|Opus)
  → fragmented MP4 (1 s fragments) via StreamTarget
  → disk.worker.ts: OPFS createSyncAccessHandle, flush every 2 s
  → on stop: finalize + copy into the user's library folder (FSA dir handle)
```

Module map: `src/capture` (device acquisition) · `src/audio` (mix graph + meters) · `src/compositor` (worker, **pure geometry in `layout.ts`**, shared renderer in `scene.ts`, code-drawn backdrops in `backdrops.ts`) · `src/recorder` (session orchestrator, encoder presets, OPFS writer, crash recovery) · `src/convert` (trim + MP4/WebM/MOV via mediabunny Conversion) · `src/enhance` (RNNoise + BS.1770 loudness, video packets pass through untouched) · `src/library` (FSA folder, scan, thumbs) · `src/pip` (Document PiP deck) · `src/state` (zustand; **live objects go in `src/recorder/runtime.ts`, never the store**) · `src/app` (screens + `controller.ts` orchestration).

## Invariants — do not break these

1. **Never record through FSA `createWritable()`.** It commits only on `close()`; a crash loses everything. Live recording writes through OPFS `createSyncAccessHandle` in `disk.worker.ts` (durable, flushed every 2 s), then `promotePartToLibrary` copies the finished file out. Crash recovery (`recovery.ts`) depends on `.part.mp4` files in OPFS.
2. **The user-gesture chain.** One click cannot both open the Document PiP deck and call `getDisplayMedia`, so the two privileged calls live on separate gestures: the preflight "Select screen" click acquires the display stream (and the live preview shows the real surface), and the "Start recording" click only opens the deck and runs the countdown. Don't merge them back into one click.
3. **Pause = `videoSource.pause()` + `audioSource.pause()` in the same microtask** (mediabunny offsets timestamps for a gapless file). Mic mute is gain = 0, never `track.stop()` — silence must keep flowing for sync. The UI clock (`accumulatedMs`) is display-only.
4. **Compositing is frame-driven, never rAF/timer-driven** (hidden tabs throttle timers to 1 Hz; capture frame pumps keep flowing). The 1 Hz heartbeat redraw exists so static screens still emit fragments.
5. **Bubble geometry has one source of truth**: `compositor/layout.ts` pure functions, consumed identically by preflight canvas, PiP deck overlay and the worker. Change the math once, everywhere follows. These are unit-tested; keep them DOM-free.
6. **Recordings are fragmented MP4; every post-op output (trim/convert/enhance/recover) is standard MP4** (`fastStart: false`) for player compatibility.
7. AAC is probed at boot (`getFirstEncodableAudioCodec`); Opus is the fallback. Never hardcode AAC (Linux CI has no AAC encoder — e2e asserts accept both).

## Invariant #8 (added after issue #4)

**Never let a constructed recording session idle.** `beginRecording()` runs the countdown FIRST, then builds the pipeline and calls `output.start()` immediately. A session constructed before the countdown (encoders + track sources waiting ~3 s for `start()`) crashes Chrome's renderer when a mic is involved. Regression-guarded by `e2e/real-flow.spec.ts`, which records in real mode (3 s countdown, real PiP, folder-mode library via an IDB-seeded handle) and fails on any `page.crash`. A `?cd=N` query param overrides the countdown length for debugging.

## Invariant #9 (added with issue #5 — scene framing)

**Scene framing lives entirely in the shared renderer, never the pipeline.** `drawScene` order is: backdrop (`compositor/backdrops.ts`, painted to fill the canvas) → inset screen/camera in a rounded, optionally shadowed frame (`screenFrameRect` + `containRect`) → bubble on top (still clamped to the **full canvas**, so it can straddle the frame edge). Three rules:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sahajamit/framecast](https://github.com/sahajamit/framecast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-23 -->
