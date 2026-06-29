---
trigger: always_on
description: React Native library for trimming video and audio. Supports both the **Old Architecture** (Bridge) and **New Architecture** (Fabric/TurboModules). Built with `react-native-builder-bob`, Yarn 4 workspaces, and TypeScript. Uses **FFmpegKit** for media processing on both platforms.
---

# AGENTS.md — react-native-video-trim

## Project Overview

React Native library for trimming video and audio. Supports both the **Old Architecture** (Bridge) and **New Architecture** (Fabric/TurboModules). Built with `react-native-builder-bob`, Yarn 4 workspaces, and TypeScript. Uses **FFmpegKit** for media processing on both platforms.

- **iOS**: Swift + Obj-C++ (`AVFoundation`, `AVKit`, `UIKit`, `Photos`, FFmpegKit)
- **Android**: Kotlin + Java (`FFmpeg-mobile`, `MediaMetadataRetriever`, `FileProvider`)
- **JS/TS**: TurboModule spec with codegen (`VideoTrimSpec`)

## Repository Layout

```
src/                          # TypeScript source — public API, TurboModule spec, old-arch bridge
  index.tsx                   # Entry point: architecture detection, factory functions, exported API
  NativeVideoTrim.ts          # TurboModule Spec (codegen source of truth for API surface & events)
  OldArch.ts                  # NativeModules fallback for Old Architecture
  __tests__/                  # Jest tests (placeholder)

ios/                          # Flat directory — all Swift/Obj-C++ native code
  VideoTrim.mm                # Dual-arch RN bridge (New Arch: NativeVideoTrimSpecBase, Old Arch: RCT_EXTERN)
  VideoTrim.swift             # Core implementation: RCTEventEmitter, FFmpeg trim, editor, file helpers
  VideoTrimProtocol.swift     # Delegate protocol for New Arch event forwarding
  VideoTrimmerViewController.swift  # Full-screen editor UI (theme, transforms, crop, player lifecycle)
  VideoTrimmer.swift          # Custom UIControl trimmer (thumbnails, handles, scrub, zoom, waveform)
  VideoTrimmerThumb.swift     # Trimmer handle visuals
  AudioWaveformView.swift     # UIView that renders audio waveform bars (rounded-rect, normalised amplitudes)
  CropOverlayView.swift       # Freeform crop overlay (brackets, grid, drag/pinch, theme-aware colors)
  AssetLoader.swift           # Async AVURLAsset loading
  ErrorCode.swift             # Error code enum
  ProgressAlertController.swift  # Modal progress UI during FFmpeg trim
  VideoTrim-Bridging-Header.h # Swift/Obj-C interop

android/
  build.gradle                # Library module: plugins, SDK versions, source sets, FFmpeg dependency
  gradle.properties           # Default SDK/NDK/FFmpeg versions
  src/main/                   # Shared base classes, UI, utilities
    java/com/videotrim/
      BaseVideoTrimModule.kt  # Core logic: editor, FFmpeg, file ops, VideoTrimListener
      VideoTrimPackage.kt     # NativeModule registration
      enums/ErrorCode.java
      interfaces/             # VideoTrimListener, IVideoTrimmerView
      utils/                  # MediaMetadataUtil, StorageUtil, VideoTrimmerUtil
      widgets/VideoTrimmerView.kt    # Full-screen trimmer UI (theme, transforms, crop, player lifecycle, waveform)
      widgets/AudioWaveformView.kt   # Custom View that renders audio waveform bars (rounded-rect, normalised amplitudes)
      widgets/CropOverlayView.kt     # Freeform crop overlay (brackets, grid, drag/pinch, theme-aware colors)
    java/iknow/android/utils/ # Screen, dp/px, background/UI thread helpers
    res/                      # Drawables, layout, colors, strings, file_paths.xml
  src/oldarch/                # Old Architecture module (ReactModule, DeviceEventEmitter)
  src/newarch/                # New Architecture module (TurboModule, codegen emitters)

example/                      # Yarn workspace example app
  src/App.tsx                 # Active demo (New Arch event listeners)
  src/App.OldArch.tsx         # Old Arch demo (NativeEventEmitter)

VideoTrim.podspec             # CocoaPods spec (reads FFMPEGKIT_PACKAGE env var)
```

## Architecture & Data Flow

### Architecture Detection

Runtime detection in `src/index.tsx`:

```typescript
const isFabric = !!(global as any).nativeFabricUIManager;
const VideoTrim = isFabric
  ? require('./NativeVideoTrim').default
  : require('./OldArch').default;
```

### Native Module Name

The module is registered as `"VideoTrim"` on both platforms and both architectures.

### Event System

Both platforms emit the same logical events: `onShow`, `onHide`, `onLoad`, `onStartTrimming`, `onFinishTrimming`, `onCancelTrimming`, `onCancel`, `onLog`, `onStatistics`, `onError`.

- **Old Architecture**: a single native event `"VideoTrim"` is emitted. The payload includes a `"name"` field with the logical event name. JS listens via `NativeEventEmitter`.
- **New Architecture**: each logical event maps to a dedicated codegen emitter (`emitOnLoad`, `emitOnFinishTrimming`, etc.) declared on the `Spec` as `EventEmitter<T>` fields.

### Factory Functions

`src/index.tsx` provides factory functions (`createBaseOptions`, `createEditorConfig`, `createTrimOptions`, `createCompressOptions`, `createFrameExtractionOptions`, `createExtractAudioOptions`, `createGifOptions`, `createMergeOptions`) that merge user overrides with defaults and run `processColor` on color string props before passing to native.

### Headless APIs


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maitrungduc1410/react-native-video-trim](https://github.com/maitrungduc1410/react-native-video-trim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
