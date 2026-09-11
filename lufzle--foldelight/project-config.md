---
trigger: always_on
description: This file gives coding agents the initial context needed to change foldelight safely. Read it before editing. Use the focused code map below instead of scanning every historical report.
---

# AGENTS.md

This file gives coding agents the initial context needed to change foldelight safely. Read it before editing. Use the focused code map below instead of scanning every historical report.

## Project identity

- Product: **foldelight**
- Creator: **Dario Farzati**
- License: **AGPL-3.0-only**
- Bundle identifier: `com.lufzle.foldelight`
- Swift package, executable, and module: `foldelight`
- Platform: macOS 14 or later
- Dependencies: Apple frameworks only
- Built app: `dist/foldelight.app`

Do not introduce the former project name or other creator attribution. Preserve copyright and SPDX headers in source files.

## Product model

foldelight reads the MacBook hinge angle and renders a live folding-glass view of the built-in display. The captured desktop stays on a fixed, fully open plane. A separate virtual glass plane follows the lid. The effect covers the complete built-in display, including the menu bar.

The key visual invariants are:

- The fold starts below the activation angle. The default is 90°.
- The virtual lid and manual preview range from 0° through 132°.
- The desktop stays visually fixed while projection through the glass changes.
- Rounded display corners exist from the first active frame through maximum fold.
- Blur grows with separation and concentrates near the lifted boundary.
- Vignette shades both sides and the lifted edge while preserving the hinge-side center.
- The final 20% of fold travel fades continuously to exact black.
- Opening uses the same geometry and fade curve in reverse.
- The effect includes the menu bar. Never add a protected top strip.
- The open state is exact neutral output.

Current defaults live in `EffectSettings`:

- Blur: `0.9`
- Vignette: `0.5` (stored internally as `shadow` for settings compatibility)
- Activation angle: `90.0`

Saved values use the app's UserDefaults domain. A code default change does not overwrite an existing user's values. `Reset effect` creates a fresh `EffectSettings` value.

## Start here

Read these files for most changes:

| Area | Primary files |
| --- | --- |
| App lifecycle, menu, window policy | `Sources/foldelight/main.swift` |
| State, persistence, commands | `Sources/foldelight/AppModel.swift` |
| Settings and About UI | `Sources/foldelight/SettingsView.swift` |
| Effect defaults and angle mapping | `Sources/foldelight/Effect.swift`, `FoldBlackout.swift` |
| HID sensor input | `Sources/foldelight/LidSensor.swift`, `LidReportSelection.swift` |
| Sparse-motion reconstruction | `Sources/foldelight/LidMotionEstimator.swift`, `LatestAngleInput.swift` |
| Screen capture and overlay | `Sources/foldelight/DesktopCapture.swift`, `DesktopOverlayPolicy.swift` |
| Live scheduling | `Sources/foldelight/LiveRenderWorker.swift`, `RenderExecutor.swift`, `MetalFrameClock.swift` |
| GPU renderer and shader | `Sources/foldelight/Renderer.swift`, `Resources/Bend.metal` |
| Miniature and interaction | `Sources/foldelight/Preview.swift`, `PreviewLidDrag.swift`, `LidAngleSlider.swift` |
| Diagnostics | `Sources/foldelight/DebugLog.swift`, `DiagnosticRecorder.swift` |
| Packaging | `Package.swift`, `Tools/build-app.sh` |
| Test and mutation commands | `TESTING.md`, `Tools/mutation-test.py` |

Check production code and current tests before relying on documentation.

## Runtime data flow

1. `LidSensor` opens the HID device, then reads it on a serial user-interactive queue.
2. Its direct sample callback feeds `DesktopCapture` before `AppModel` receives throttled UI telemetry.
3. `DesktopCapture` captures native ScreenCaptureKit frames into a latest-value mailbox.
4. Sensor or capture changes wake `LiveRenderWorker`.
5. `LidMotionEstimator` predicts only within bounded age, velocity, lead, and offset limits.
6. `LiveRenderWorker` requests drawables from the display clock and submits Metal work through `RenderExecutor`.
7. `BendRenderer` updates the compact blur pyramid only when source damage and effect state require it.
8. The overlay appears only while an effect frame is needed. It stays excluded from its own capture stream.

Do not move HID polling, capture callbacks, GPU submission, or per-frame work onto the main thread. UI telemetry is intentionally slower than the direct renderer path.

## Sensor rules

Apple does not document the report layouts used here. Report 1 supplies whole degrees. Report 7 can supply hundredths only when `PreciseLidCapability` matches the exact vendor, product, and descriptor. `LidReportSelection` brackets a fine read with coarse reads and requires agreement. A failed fine read falls back to report 1 in the same call and disables precision until reconnect.

Three consecutive read failures stop polling. `Reconnect lid sensor` closes and reopens the device, resets failure and report-selection state, and resumes polling. Keep the recovery action unless the sensor lifecycle changes.

## Capture and overlay rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lufzle/foldelight](https://github.com/lufzle/foldelight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
