---
trigger: always_on
description: Guidance for AI coding agents (and humans) working in this repository. **Read this before making
---

# AGENTS.md — CameraX

Guidance for AI coding agents (and humans) working in this repository. **Read this before making
changes.** It describes the rules, architecture, design system, and shared components.

Play Store camera app (`com.arindam.camerax`) built with Jetpack CameraX 1.6.1. Kotlin, Jetpack Compose, minSdk 23, compileSdk/targetSdk 37. Goal: show other apps what the CameraX library can do.

## Architecture (Clean Architecture, single module)

```
ui (presentation) → domain ← data
```

Do not construct `CameraSession` from the UI. Keep use cases; do not collapse to ViewModel → CameraSession. Do not split Gradle modules unless asked.

Disk and MediaStore work (`list` / `delete` / `publish` / `stitch` / motion mux / still effects) must stay off the main thread: repositories use `AppDispatchers.io`, ViewModels `withContext`. Do not call `listFiles`, `BitmapFactory.decodeFile`, or `MediaMetadataRetriever` from Compose or click handlers.

### File map (where to change what)

| Goal | Start here |
|---|---|
| Add a pager mode (Photo/Video/…) | `domain/model/CameraModeCatalog.kt` + `CameraMode.labelRes` in `ui/home/camera/CameraModels.kt`. New CameraX session only in `CameraSession` if bind flags are not enough. |
| Bind / capture / zoom / flash / AE | `CameraViewModel` → `domain/usecase` → `CameraRepository` → `data/camera/CameraSession.kt` |
| Live-feed chrome (header, shutter, zoom) | `ui/home/camera/CameraChrome.kt` |
| Mode pager physics | `ui/home/camera/CameraPager.kt` |
| Settings row | `settingsSections()` in `ui/settings/SettingsCatalog.kt`; capabilities in `SettingsViewModel` |
| Theme (Light/Dark/System) | `util/theme/NightMode.kt` + `SettingsActivity` |
| Motion Photo mux / gallery play | `data/camera/MotionPhotoMuxer.kt` |
| Panorama stitch | `data/camera/PanoramaStitcher.kt` |
| Publish to DCIM | `data/media/MediaStorePublisher.kt` via `PublishMedia` |
| Composition root | `di/AppContainer.kt` (`CameraInteractors`) |
| Shared glass chrome (back, pills) | `ui/compose/CameraGlassButton.kt` (`ChromeControlSize`), `ChromeActionPill.kt` |

`CameraFragment` is a thin Compose host only (plus IMAGE_CAPTURE / MOTION_PHOTO result delivery). Do not resurrect the old View-based capture/flash/zoom code. Public types have KDoc describing which layer they belong to.

## CameraX rules

- Bind `Preview + ImageCapture + VideoCapture` together when possible; fall back (drop video, stills only) on `IllegalArgumentException`. Slow-motion (`CameraMode.SLOW_MOTION`) uses `HighSpeedVideoSessionConfig` (`Preview + VideoCapture` only, no audio, no ImageCapture / extensions / color `CameraEffect`). Hide Slo-mo from the mode pager if `Recorder.getHighSpeedVideoCapabilities` is null / has no SDR qualities; probe that at ViewModel start so the chip is not delayed until bind. Settings still show slo-mo quality and capture fps; disable those rows when the device lists none.
- Rebind only for lens, physical back cameras (ultra-wide/tele), OEM extensions, enabling/disabling the color `CameraEffect`, night-scene auto-switch (OEM Night), Settings photo aspect / video quality / video stabilization / Ultra HDR / RAW / DNG / full-sensor RAW / slo-mo quality and fps / 60 fps (`GroupableFeature.FPS_60`), or entering/leaving slow-motion or Dual. Zoom, torch, flash, effect matrix, target rotation, hybrid AE, and exposure compensation must not rebind. Extra back cameras are listed from `availableCameraInfos` + focal length and shown as 0.5x/1x/2x chips that rebind by camera id. Flip during a normal video recording is allowed when Settings **Flip while recording** is on (default off): start with `PendingRecording.asPersistentRecording()`, keep the same `VideoCapture`/`Recorder`, `unbindAll()`, and bind the new lens — do not stop the clip. Skip persistent recording for slow-motion, Dual, and motion photos.
- Dual mode binds concurrent front+back `Preview` via `availableConcurrentCameraInfos` (hide Dual if empty). Prefer concurrent stills; fall back to preview-only if `ImageCapture` cannot bind. Do not add ML Kit / face overlays. `ImageAnalysis` is used only in Effects mode for live ColorMatrix effects.
- OEM extension chips (HDR / Night / Portrait=`BOKEH` / Beauty=`FACE_RETOUCH`) only if `ExtensionsManager.isExtensionAvailable`. Extensions typically cannot bind with `VideoCapture`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arindamxd/camerax-android](https://github.com/arindamxd/camerax-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
