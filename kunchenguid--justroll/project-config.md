---
trigger: always_on
description: This file provides guidance to agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to agents when working with code in this repository.

## What this is

`justroll` is a macOS CLI that records every selected screen and camera to its own clean file in one command.
Each clip carries a copy of the same microphone track, so any editor's "sync by audio" lines them up without timecode gear.
It is a single-package ESM Node project (`"type": "module"`, Node >= 20) using **ink** + **React** (via `htm`, not JSX) for the terminal UI, and **ffmpeg** as the only runtime dependency.

## Commands

```sh
pnpm install               # uses pnpm (packageManager pinned); pnpm-workspace.yaml present
pnpm test                  # node --test over test/*.test.js
node --test test/plan.test.js          # run a single test file
node --test --test-name-pattern "groups all screens"   # run one test by name
pnpm run lint              # eslint src bin test scripts
pnpm run format            # prettier --write
pnpm run format:check      # prettier --check (CI uses this)
pnpm run demo              # live UI with a synthetic engine, records nothing
pnpm run selftest          # headless real-device capture that verifies the pipeline
make demo                  # regenerate demo.gif/demo.mp4 (needs vhs + ffmpeg)
```

Per Kun's global instructions: use TDD for features and bug fixes; tests live in `test/` mirroring `src/` module names.

## Architecture

The codebase is split into **pure, testable logic** (most of `src/`) and **side-effecting edges** (process spawning in `recorder.js`, device enumeration in `devices.js`, the ink UI, and `bin/justroll.js`). Almost everything that builds an ffmpeg command, a path, or a UI decision is a pure function so it can be unit-tested without a device.

### The recording pipeline (data flow)

1. **`bin/justroll.js`** — entry point. Parses argv, loads config, checks ffmpeg, enumerates devices, then renders the ink `App` (or runs `--selftest`/`--demo` headless/synthetic paths). Holds the CLI flags (`--dir`, `--no-mp4`, `--fps`, `--seconds`, `--selftest`, `--demo`).
2. **`devices.js`** — `enumerateDevices()` shells `ffmpeg -list_devices` and `parseDeviceList()` parses its stderr into `{ video:[{index,name,kind}], audio }`. `kind` is `screen` vs `camera` by name match. Indexes drift on replug, so devices are remembered by **name** and re-resolved via `resolveDeviceIndex`. **Camera/capture-card framerates** are mode-locked (a card may only do `1080p60`), so `probeVideoModes()` provokes ffmpeg's "Supported modes" listing (by requesting an impossible 1 fps) and `pickFramerate()` picks a supported rate; an empty list means the device won't open. **Never probe a screen this way** — a screen might actually start a 1 fps capture and hang.
3. **`plan.js`** — `buildPlan()` turns wizard selections + config into a fully-resolved plan: every directory, ffmpeg input index, output filename, and label decided **up front**. This is the contract every downstream piece consumes. `ensurePlanDirs()` creates `dir/{raw,exports,project}`.
4. **`naming.js`** — pure title→filesystem naming: `slugify`, `sessionDirName` (`YYYY-MM-DD_slug`), `uniqueDirName` (collision suffixing), `assignLabels` (`screen-0`, `screen-1`, `camera`/`camera-0`).
5. **`ffmpegArgs.js`** — **pure** ffmpeg argv builders, the heart of the tool. `buildJobArgs` (grouped **video-only** job), `buildAudioRecordArgs` (the mic's own isolated PCM file), `buildRecordArgs` (single source, legacy/tests), `buildAudioTapArgs` (PCM tap for the meter), `buildRemuxArgs` (MKV→MP4), and `parseProgress`/`normalizeProgress` for `-progress pipe:1`. Side-effect free — never spawns.
6. **`recorder.js`** — the only place processes are spawned. `Recorder` (an `EventEmitter`) runs the plan as **independent processes**: video-only capture jobs + the mic's own `buildAudioRecordArgs` process + a decoupled `startMicTap` for the live waveform. It polls per-file byte sizes from the filesystem + fps/drop from each progress stream, estimates each stream's start wall-clock from `-progress` (`_recordT0`) for cross-file `startOffsetMs` alignment, and tears down cleanly on `stop()` (`q` to stdin → SIGINT → SIGKILL), then remuxes video to MP4. `FfmpegEngine` is the **pluggable engine seam** (`createRecording(plan)`); `MockEngine` and a future `ObsEngine` implement the same surface.
7. **`session.js`** — writes the artifacts: `session.json` manifest (`buildSessionManifest`) and human-readable `notes.md` sync recipe (`buildNotesMarkdown`).
8. **`thumbnail.js`** — **pure** pixel→terminal rendering. `renderRgbHalfBlocks()` turns a raw RGB24 frame into half-block (`▀`) lines with 24-bit ANSI color (top pixel = foreground, bottom = background, doubling vertical resolution); `syntheticThumbnail()` fakes one for `--demo`. Paired with `buildScreenThumbnailArgs` (ffmpegArgs) + `grabScreenThumbnail` (recorder) to show **per-device previews of both the cameras and screens steps**, so identical monitors (or an ambiguous capture card) are told apart by their actual content, not a useless "Capture screen N" label. Camera grabs use the device's probed framerate.

### The macOS screen-capture invariant (critical, do not break)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kunchenguid/justroll](https://github.com/kunchenguid/justroll) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
