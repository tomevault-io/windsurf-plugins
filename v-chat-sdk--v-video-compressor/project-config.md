---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**v_video_compressor** is a professional Flutter plugin for high-quality video compression with real-time progress tracking and thumbnail generation. The plugin uses native platform APIs (Media3 for Android, AVFoundation for iOS) rather than ffmpeg for optimal performance and smaller app size.

**Version**: 1.2.1
**Platforms**: Android (API 21+), iOS (11.0+)
**Key Features**: Multi-quality compression, real-time progress, global progress stream, thumbnail generation, batch processing

## Architecture

### Plugin Structure (Platform Channels)

The plugin follows Flutter's platform channel pattern with three layers:

1. **Dart Layer** (`lib/`):
   - `v_video_compressor.dart` - Main API facade with logging and validation
   - `v_video_compressor_platform_interface.dart` - Abstract platform interface
   - `v_video_compressor_method_channel.dart` - MethodChannel implementation
   - `src/v_video_models.dart` - All data models, enums, and configurations
   - `src/v_video_stream_manager.dart` - Global progress event stream manager (v1.2.0+)
   - `src/v_video_logger.dart` - Comprehensive logging system

2. **Android Native** (`android/src/main/kotlin/`):
   - `VVideoCompressorPlugin.kt` - MethodChannel handler and lifecycle management
   - `VVideoCompressionEngine.kt` - Core compression using Media3, handles progress calculation
   - `VVideoMediaLoader.kt` - Video metadata extraction
   - `VVideoModels.kt` - Kotlin data classes mirroring Dart models

3. **iOS Native** (`ios/Classes/`):
   - `VVideoCompressorPlugin.swift` - MethodChannel handler
   - `VVideoCompressionEngine.swift` - Core compression using AVFoundation
   - `VVideoMediaLoader.swift` - Video metadata extraction
   - `VVideoModels.swift` - Swift data classes

### Progress Tracking Architecture

**v1.2.0+** introduced a global progress stream accessible from anywhere:

- **Dart**: `VVideoStreamManager` manages EventChannel → StreamController broadcast
- **Native**: Platforms emit progress events via `v_video_compressor/progress` EventChannel
- **Event Flow**: Native → EventChannel → StreamController → Multiple Listeners
- **Type Safety**: All events use typed `VVideoProgressEvent` model (no raw Maps)

### Key Design Patterns

- **Platform Interface Pattern**: Abstract `VVideoCompressorPlatform` with MethodChannel implementation
- **Singleton Stream Manager**: Global progress stream accessible via static methods
- **Broadcast Stream**: Multiple widgets/services can listen to progress simultaneously
- **Facade Pattern**: `VVideoCompressor` class provides simple API over complex platform logic
- **Data Transfer Objects**: All communication uses immutable data classes with `toMap()`/`fromMap()`
- **Validation at Boundaries**: Dart validates configurations before platform call
- **Comprehensive Logging**: Optional detailed logging with configurable levels

## Common Development Commands

### Flutter/Dart Commands

```bash
# Get dependencies
flutter pub get

# Analyze code (DO NOT build APK or run on device)
flutter analyze

# Run tests
flutter test

# Format code
dart format .

# Check for lints
flutter analyze

# Run example app (if needed for testing)
cd example
flutter pub get
flutter run
```

### Android-Specific

```bash
# Compile Android code only (no APK build)
cd example/android
./gradlew assembleDebug

# Check for Android issues
./gradlew lintDebug
```

### iOS-Specific

```bash
# Install pods
cd example/ios
pod install

# Compile iOS code (requires macOS)
cd ../..
flutter build ios --no-codesign
```

### Platform Testing

**IMPORTANT**: For development speed, use `flutter analyze` instead of building APKs or running on devices. Full builds are time-consuming and only needed for final validation.

## Code Modification Guidelines

### When Adding New Features

1. **Check Existing Implementation First**: Search for similar features in both Dart and native code
2. **Update All Three Layers**: Dart interface → Android implementation → iOS implementation
3. **Mirror Data Models**: Keep Dart, Kotlin, and Swift models in sync
4. **Validate Configurations**: Add validation to `isValid()` methods in model classes
5. **Add Logging**: Use `VVideoLogger` for all method calls and important events
6. **Document Public APIs**: Add comprehensive dartdoc comments

### Platform-Specific Compression Logic

**Android** (`VVideoCompressionEngine.kt`):
- Uses Media3 `Transformer` API
- Hardware acceleration via `DefaultEncoderFactory`
- Progress calculation: estimated based on bitrate and duration
- Output: MP4 with H.264/H.265 video, AAC audio

**iOS** (`VVideoCompressionEngine.swift`):
- Uses AVFoundation `AVAssetExportSession`
- Quality presets: `.AVAssetExportPresetHighestQuality`, `.AVAssetExportPreset1920x1080`, etc.
- Progress via `exportSession.progress` KVO observation
- Output: MP4 with H.264/HEVC video, AAC audio

### Global Progress Stream (v1.2.0+)

When working with progress tracking:

- **Dart Side**: Events flow through `VVideoStreamManager.progressStream`
- **Native Side**: Emit events via EventChannel `v_video_compressor/progress`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [v-chat-sdk/v_video_compressor](https://github.com/v-chat-sdk/v_video_compressor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
