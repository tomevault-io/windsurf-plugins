---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build

```bash
make build      # Debug build
make release    # Release build
make dev        # Build debug and run
make run        # Build release and run
make dmg        # Create DMG installer
make install    # Install to /Applications
make format     # Format Swift source (swift format)
make clean      # Clean build artifacts
make tag        # Create git tag from Config.xcconfig version and generate changelog
make changelog  # Generate CHANGELOG.md
```

No test target exists yet. No linter is configured.

## Architecture

Reframed is a macOS screen recording app with a menu bar interface, floating capture toolbar, built-in video editor, and `.frm` project management (macOS 15+, Swift 6 strict concurrency).

### Source layout

```
Reframed/
├── App/              AppDelegate, Permissions, WindowController
├── CaptureModes/     Area/Screen/Window/Device selection + Common overlay components
├── Compositor/       Video composition & export (VideoCompositor, FrameRenderer, ExportSettings, BackgroundStyle, CameraLayout, GradientPresets)
├── Editor/           Video editor (timeline, properties panels, preview, cursor, zoom, camera regions, history)
├── Libraries/        Native C/C++ dependencies (gifski static library for GIF encoding)
├── Logging/          LogBootstrap, RotatingFileLogHandler
├── Project/          .frm bundle management (ReframedProject, ProjectMetadata)
├── Recording/        Capture pipeline (coordinators, writers, device/webcam/mic/audio, cursor metadata)
├── State/            SessionState, CaptureState, CaptureMode, ConfigService, StateService, RecordingOptions, KeyboardShortcutManager
├── UI/               Toolbar, menu bar, popovers, settings, countdown overlay, reusable components
├── Utilities/        CGRect/NSScreen extensions, CodableColor, SendableBox, SoundEffect, TimeFormatting, UpdateChecker, MediaFileInfo, RNNoiseProcessor, CIImageExtensions, KeyboardShortcut
├── ReframedApp.swift Entry point (@main)
└── Info.plist        App configuration
```

### Concurrency model

Everything is actor-isolated. The core pattern:

- **`SessionState`** (@MainActor, @Observable) — central state manager that owns all coordinators, windows, and drives the full recording lifecycle; SwiftUI binds directly to this
- **`RecordingCoordinator`** (actor) — owns `ScreenCaptureSession` + all track writers, drives the capture pipeline
- **`VideoTrackWriter`** / **`AudioTrackWriter`** (actors) — AVAssetWriter track wrappers
- **`ScreenCaptureSession`** (class, `@unchecked Sendable`) — SCStream wrapper; unchecked because SCStream isn't Sendable yet
- **`SelectionCoordinator`** (@MainActor) — manages the full-screen overlay window for area selection and recording borders
- **`WindowSelectionCoordinator`** (@MainActor) — manages window highlight overlay for window selection
- **`EditorState`** (@MainActor, @Observable) — editor state including trim ranges, background, camera layout/regions, cursor, zoom, animations, audio
- **`VideoCompositor`** (enum with static methods) — export-time compositing pipeline (manual + parallel export modes)
- **`FrameRenderer`** (NSObject, AVVideoCompositing) — custom compositor for rendering background + screen + webcam with camera region transitions

### State machine

```
idle → selecting → countdown(remaining) → recording(startedAt) ⇄ paused(elapsed) → processing → editing → idle
```

### Recording flow

Reframed offers four recording modes:

- **Entire screen** — captures the full display
- **Selected window** — window highlight overlay, captures a single application window
- **Selected area** — full-screen transparent overlay with crosshair cursor; drag to select region (8 resize handles)
- **iOS device** — captures connected iPhone/iPad via AVCaptureDevice

Optional features: webcam PiP overlay (with hide-while-recording option), microphone audio, system audio capture, configurable countdown timer (3/5/10s), cursor metadata recording with mouse click monitoring.

After selection, ScreenCaptureKit captures the chosen target. CVPixelBuffers flow through `SharedRecordingClock` for timestamp synchronization, then to track writers. On stop, a `.frm` project bundle is created and the editor opens automatically.

### Video editor

Built-in editor with:
- Timeline trimming (independent trim ranges for video, system audio, mic audio)
- Audio regions (per-track independent audio trimming with volume and mute controls)
- Background styles (none, solid color, gradient presets, background image with fill modes)
- Canvas aspect ratios (original, 16:9, 1:1, 4:3, 9:16)
- Padding and corner radius (both video and camera)
- Webcam PiP (draggable positioning, 4-corner presets, configurable size/corner radius/border/shadow/mirror)
- Camera regions (timeline-based webcam visibility: fullscreen/hidden/custom with per-region entry/exit transitions — fade/scale/slide)
- Cursor overlay (multiple styles, smoothing levels, click highlights with configurable color/size)
- Cursor movement smoothing (spring physics-based interpolation with speed presets)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jkuri/Reframed](https://github.com/jkuri/Reframed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
