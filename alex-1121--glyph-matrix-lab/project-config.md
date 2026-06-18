---
trigger: always_on
description: This file provides guidance to coding agents working in this repository.
---

# AGENTS.md

This file provides guidance to coding agents working in this repository.

## Build Commands

```bash
./gradlew assembleDebug        # Build debug APK
./gradlew assembleRelease      # Build release APK
./gradlew installDebug         # Build and install on connected device
./gradlew test                 # Run unit tests
./gradlew testDebugUnitTest    # Run debug JVM tests only (fast local loop)
./gradlew connectedAndroidTest # Run instrumented tests (requires device)
./gradlew clean build          # Clean rebuild
```

## Testing

Use these commands depending on what you need to verify:

```bash
./gradlew testDebugUnitTest
```

Runs the fast local JVM test suite for the debug variant. Use this for extracted pure logic such as `PixelGrid`, `FrameBuilders`, and `EqualizerProcessor`.

```bash
./gradlew test
```

Runs the full local unit test suite across configured variants.

```bash
./gradlew connectedAndroidTest
```

Runs instrumented tests on a connected device or emulator. This is slower and requires hardware/device setup.

For toy behavior changes, do not stop at Gradle tests. Also verify on-device:

```bash
./gradlew installDebug
adb shell am start -n com.nothing.thirdparty/com.nothing.thirdparty.matrix.toys.manager.AodToySelectActivity
```

Then reselect the toy in `Settings -> Glyph Interface -> Always-on Glyph Toy` and confirm the rendered behavior on the phone.

## Device Setup

Debug mode is required and auto-disables after 48 hours:

```bash
adb shell settings put global nt_glyph_interface_debug_enable 1
```

Open Toys Manager on device:

```bash
adb shell am start -n com.nothing.thirdparty/com.nothing.thirdparty.matrix.toys.manager.ToysManagerActivity
```

Or via intent in code:

```kotlin
val intent = Intent()
intent.setComponent(
    ComponentName(
        "com.nothing.thirdparty",
        "com.nothing.thirdparty.matrix.toys.manager.ToysManagerActivity"
    )
)
startActivity(intent)
```

Open the Always-on Glyph Toy picker directly:

```bash
adb shell am start -n com.nothing.thirdparty/com.nothing.thirdparty.matrix.toys.manager.AodToySelectActivity
```

Device navigation path:

```text
Settings -> Glyph Interface -> Always-on Glyph Toy
```

Only one toy can be active at a time. The master toggle is at the top of that screen.

## Project Overview

This is an Android app for creating Glyph Toys, which are services that render visuals on the Nothing Phone (4a) Pro 13x13 Glyph Matrix LED display. The launcher UI handles audio permission, shows the latest live toy frame when available, lists saved custom images, and provides a masked 13x13 image editor. Toy output still comes from background services activated from device Settings.

SDK location:

```text
app/libs/glyph-matrix-sdk-2.0.aar
```

The proprietary Nothing SDK provides `GlyphMatrixManager`, `GlyphMatrixFrame`, `GlyphMatrixObject`, `GlyphToy`, and related classes under `com.nothing.ketchum`.

Preview assets for toy registration should be prepared as SVG/vector drawables where possible; the Nothing README explicitly recommends exporting preview images as SVG before importing them into Android Studio.

If a toy needs real-time output audio analysis via `android.media.audiofx.Visualizer` on session `0`, declare `android.permission.RECORD_AUDIO` in the manifest and request it at runtime from the launcher activity; without it, `Visualizer(0)` may fail and the toy should degrade gracefully.

## Target Device

- Device identifier: `Glyph.DEVICE_25111p`
- Matrix size: 13x13 LEDs
- No Glyph Touch hardware
- AOD toys only
- Every toy must declare `aod_support=1` in the manifest
- `GlyphToy.EVENT_AOD` fires every minute while the toy is active

## Architecture

### Creating a Toy

Each toy is an Android `Service` that extends `GlyphToyBase`. The base class:

- Binds to `GlyphMatrixManager` in `onCreate`
- Unbinds in `onDestroy`
- Routes IPC messages from the Glyph system using `Messenger` and `Handler`
- Exposes callbacks: `onServiceConnected(context, gmm)`, `onServiceDisconnected(context)`, `onAod()`, `onTouchDown()`, `onTouchUp()`, and `onLongPress()`

### Pure Domain Layer

Toys can delegate to pure abstractions that are testable without Robolectric:

- `PixelGrid` — 13×13 boolean domain model (no Android dependencies)
- `FrameSink` — interface for rendering frames
- `GlyphDisplayAdapter` — converts `PixelGrid` → `Bitmap` → SDK
- `FrameBuilders` — static builders for clock, equalizer, call grids
- `EqualizerProcessor` — waveform → bar heights with decay
- `CompositeToyController` — orchestrates mode switching (CALL/EQUALIZER/CUSTOM_IDLE/CLOCK)
- `CustomGlyphProvider` / `RepositoryCustomGlyphProvider` — supplies selected idle custom images to the composite toy
- `GlyphImageSerializer` — serializes custom 13x13 images as 169-character binary strings

This separation enables fast JVM unit tests via `./gradlew testDebugUnitTest`.

### App UI And Preview

- `MainActivity` requests microphone permission, shows the latest in-process `LiveGlyphPreview` frame, and falls back to the configured custom image when no live frame exists.
- `LiveGlyphPreview` is process-local memory. Force-stopping or reinstalling the app clears the latest frame until an active toy service publishes again.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alex-1121/glyph-matrix-lab](https://github.com/alex-1121/glyph-matrix-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
