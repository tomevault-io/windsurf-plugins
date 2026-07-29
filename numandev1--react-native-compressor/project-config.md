---
trigger: always_on
description: `react-native-compressor` is a React Native library that compresses Image, Video, and Audio (WhatsApp-style auto compression, plus manual mode), with background upload/download and video thumbnail helpers. The TypeScript/JS layer is a thin wrapper over a single native module named `Compressor` implemented natively on Android (Kotlin) and iOS (Swift). It is published to npm as a library — the `examples/` apps exist only to develop and test against it.
---

# CLAUDE.md — react-native-compressor

## Project Overview

`react-native-compressor` is a React Native library that compresses Image, Video, and Audio (WhatsApp-style auto compression, plus manual mode), with background upload/download and video thumbnail helpers. The TypeScript/JS layer is a thin wrapper over a single native module named `Compressor` implemented natively on Android (Kotlin) and iOS (Swift). It is published to npm as a library — the `examples/` apps exist only to develop and test against it.

## Tech Stack & Architecture

**JS layer:** TypeScript · single native module `Compressor` exposed as a **Nitro HybridObject** (`react-native-nitro-modules`), resolved in `src/Main.tsx` via `NitroModules.createHybridObject`
**Android native:** Kotlin · hand-rolled MediaCodec/MediaMuxer video transcoder
**iOS native:** Swift (C++/Swift interop) · AVFoundation via vendored `NextLevelSessionExporter.swift`
**Codegen:** Nitrogen (`yarn nitrogen`) generates the native bindings from the `*.nitro.ts` spec into `nitrogen/generated/` (committed to git)
**Tooling:** Yarn 4 (Berry) workspace (`examples/*`) · Node `>= 22.11` · Jest (native mocked) · react-native-builder-bob · Expo config plugin
**Requirements (Nitro):** RN ≥ 0.75 · iOS ≥ 13.4 / Xcode ≥ 16.4 · Android compileSdk ≥ 34 · C++20. Works on both old & new architecture (Nitro handles its own linking)

### JS wrapper → single native module

All native functionality is exposed through one Nitro HybridObject named `Compressor`. `src/Main.tsx` resolves it once via `NitroModules.createHybridObject<Compressor>('Compressor')`, typed by the spec `src/specs/Compressor.nitro.ts`, and re-throws a friendly linking error if Nitro can't find it. Options are passed as Nitro `AnyMap` (untyped maps), parsed natively as before.

The public API is assembled in `src/index.tsx` from four domain modules plus utils:

- `src/Image/index.tsx` — `Image.compress` (strips base64 data-URI headers before calling native)
- `src/Video/index.tsx` — `Video.compress`, `cancelCompression`, `activate/deactivateBackgroundTask`
- `src/Audio/index.tsx` — `Audio.compress`
- `src/utils/` — `Uploader.tsx` (`backgroundUpload`, `cancelUpload`), `Downloader.tsx` (`download`), `helpers.ts`, and metadata/path helpers (`getRealPath`, `getVideoMetaData`, `getImageMetaData`, `generateFilePath`, `createVideoThumbnail`, `clearCache`, `getFileSize`)

### Progress is delivered via callbacks, not events

Nitro has no `NativeEventEmitter`. Progress is delivered through **callback functions passed as method parameters** (`onProgress`, `onDownloadProgress`, `onExpired`) — first-class, reference-counted, auto-scheduled onto the JS thread. Callbacks can't live inside an `AnyMap`, so any callback that used to be nested in the options object is lifted to a top-level method parameter (the JS layer strips functions/`undefined` from option maps via `toNativeOptions` in `src/utils/helpers.ts`, since Nitro's AnyMap throws on those).

A `uuid` (`uuidv4`) is still generated in JS and threaded inside the options map, but now only for (a) cancellation (`cancelCompression`, `cancelUpload`, `AbortController`) and (b) routing native progress emissions to the correct callback. Natively, the per-domain code still calls `EventEmitterHandler.emit*`, but that class is now a **uuid → callback registry** (not a bridge emitter): the binding registers the JS callback under the uuid before invoking the domain method and unregisters when the Promise settles. Keep the uuid threading consistent across JS and both native sides.

### Native code organization (mirrors the JS domains)

The thin Nitro binding lives separately from the heavy domain logic:

- **Android binding** `android/src/main/java/com/margelo/nitro/compressor/HybridCompressor.kt` (extends the generated `HybridCompressorSpec`) converts `AnyMap` → `ReadableMap`, bridges the Nitro `Promise` to the domain layer's `com.facebook.react.bridge.Promise` via `NitroPromiseAdapter.kt`, and runs domain work on a background executor. `NitroCompressorPackage.kt` (a `BaseReactPackage`) exists only so RN autolinking registers the Gradle project and its `companion init` loads `libNitroCompressor.so`.
- **iOS binding** `ios/HybridCompressor.swift` (implements `HybridCompressorSpec`) converts `AnyMap` → `NSDictionary`, synthesizes `RCTPromiseResolveBlock`/`RejectBlock` to drive the Nitro `Promise`.

Heavy domain logic (unchanged, mirrors the JS domains):

- **Android** `android/src/main/java/com/reactnativecompressor/` → `Image/`, `Video/`, `Audio/`, `Utils/`. The video transcoder is hand-rolled under `Video/VideoCompressor/` (MediaCodec/MediaMuxer pipeline: `Compressor.kt`, `MP4Builder.kt`, surfaces/renderer, `utils/`). `VideoMain.compress` routes to auto vs manual via `VideoCompressorHelper`. `StreamableVideo.kt` moves the `moov` atom to the front of the output by default — preserve this behavior.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [numandev1/react-native-compressor](https://github.com/numandev1/react-native-compressor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
