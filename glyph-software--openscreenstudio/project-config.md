---
trigger: always_on
description: Guidance for AI coding agents working on **OpenScreen Studio** — an open-source macOS clone of [screen.studio](https://screen.studio).
---

# AGENTS.md

Guidance for AI coding agents working on **OpenScreen Studio** — an open-source macOS clone of [screen.studio](https://screen.studio).

## What this is

Tauri 2 desktop app. React 19 + TypeScript + Vite frontend, Rust backend. There is **no single React state machine** — `App.tsx` routes purely by the native window label:

- **permissions** (720×760) — onboarding screen shown when screen-recording or accessibility is missing.
- **hud** (900×75 idle, 560×64 during the countdown pill, 260×52 while recording) — small frameless always-on-top bar; its internal `HudPhase` is `idle | countdown | recording`. The ✕ button quits the app via `quit_app` (unless the editor window is visible, in which case it only hides the HUD).
- **editor** (1440×900, min 1100×700) — the full editor; hidden until a recording finishes.
- **picker-\*** — dynamically-created transparent overlay windows (one per display) used to pick a display / window / drag-area before recording.
- **camera-preview** — floating live webcam preview shown while the HUD's camera toggle is on; rendered via `getUserMedia` in `CameraPreview.tsx` (the device is matched by label, not deviceId).

On launch the Rust `setup` hook reads `check_permissions` (`CGPreflightScreenCaptureAccess()` for screen recording, `AXIsProcessTrusted()` for accessibility — the latter is required for cursor tracking). If either is missing the permissions window is shown, otherwise the hud is. macOS only (entitlements + Info.plist target macOS 13+; ScreenCaptureKit recording path needs macOS 15+).

The UI was ported from a Claude Design handoff bundle. **Capture is real and in-process:**

- **Screen recording: ScreenCaptureKit**, entirely in-process via the `screencapturekit` crate (`macos_15_0` feature). An `SCStream` + `SCRecordingOutput` writes mp4 directly. Supports display, window, and cropped-area capture, plus pause / resume / restart / cancel (pause/restart segments are concatenated with the bundled ffmpeg on stop).
- **AV sidecar tracks.** System audio and microphone are captured through the same SCK stream (`SCStreamOutputType::Audio` / `::Microphone`) and written to sidecar WAVs; the camera records to a sidecar movie via `camera.rs` (AVFoundation `AVCaptureDevice` by uniqueID). The artifact carries `systemAudioPath` / `cameraPath` (+ offsets) alongside the screen mp4. All three sources are **off by default** on every launch; only the chosen device ids persist (localStorage). Each HUD toggle checks its TCC permission at toggle time.
- **Device enumeration is native** — microphones via the SCK bridge's `AudioInputDevice::list()`, cameras via `camera::list_camera_devices()`. No ffmpeg parsing.
- **Cursor sidecar.** During recording a `CursorTrack` polls cursor position, clicks, and cursor-*shape* transitions; on stop it writes a `<recording>.cursor.json` next to the mp4 (`cursorSidecarPathFor`). The editor consumes this for auto-zoom-on-click.
- **Mic-level metering:** `cpal` default input, peak amplitude emitted as `mic-level` events at ~30 Hz.
- **Artifact handoff:** `open_editor_with_artifact` shows the editor window, hides the hud, and emits `recording-artifact`; the editor loads the mp4 in a `<video>` via `convertFileSrc` (asset protocol scoped to `$HOME/Movies/**`, `$TEMP/**`, and `/System/Library/Desktop Pictures/**`).
- **External stop.** The macOS menu-bar "stop" pill ends the SCK stream out-of-band; the app gets a `recording-stopped-externally` event and runs `finalize_external_stop` to produce the same artifact.

Recordings land in `~/Movies/OpenScreen Studio/OpenScreen-<timestamp>-<uuid>.mp4` (+ matching `.cursor.json`, and audio/camera sidecar files when those sources were on).

**ffmpeg is bundled, not a `$PATH` dependency.** `scripts/fetch-ffmpeg.sh` runs on `postinstall` and downloads `src-tauri/binaries/ffmpeg-{aarch64,x86_64}-apple-darwin`; `tauri.conf.json` ships it as `externalBin: ["binaries/ffmpeg"]`. It is used for segment concat (pause/restart), muxing, and encoding exported frames — not for device enumeration or the live recording path.

**macOS permissions:** because recording is in-process via ScreenCaptureKit, the TCC screen-recording grant attaches to the **OpenScreen Studio.app bundle** (or the dev binary in `bun run tauri dev`), not to ffmpeg. Grant Screen Recording once in System Settings → Privacy & Security. Accessibility is also requested — cursor/click tracking needs it. `check_permissions` runs the screen-recording probe in a fresh subprocess because `CGPreflightScreenCaptureAccess()` caches its answer per-process.

## Toolchain

This project uses **Bun**, not npm/pnpm/yarn. Always use `bun` and `bun run`.

**Never use `npm`, `npx`, `pnpm`, `yarn`, or `pnpx`** — not even for one-off commands like `tsc` or `vite`. Use `bunx` for binaries and `bun run <script>` for package.json scripts. This applies to ad-hoc verification too (e.g. always `bunx tsc --noEmit`, never `npx tsc --noEmit`).

```sh
bun install            # install JS deps; postinstall fetches the ffmpeg binaries
bun run tauri dev      # run the desktop app (frontend + Rust)
bun run tauri build    # produce .app + .dmg in src-tauri/target/release/bundle/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Glyph-Software/OpenScreenStudio](https://github.com/Glyph-Software/OpenScreenStudio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
