---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Kompressor** is a Kotlin Multiplatform (KMP) library for compressing images, videos, and audio on Android and iOS using native hardware encoders (zero binary overhead). Published as `co.crackn:kompressor` on Maven Central (Maven group `co.crackn`; the Kotlin package remains `co.crackn.kompressor.*`).

Image, audio, and video compression are all implemented on both platforms:
- **Image**: Android `BitmapFactory` / iOS `UIImage` + Core Graphics.
- **Audio**: Android **Media3 `Transformer` 1.10** (audio-only export via `setRemoveVideo`, `SonicAudioProcessor` for resampling, `ChannelMixingAudioProcessor` for mono/stereo, AAC-in-M4A output, passthrough fast path when input is already AAC at target bitrate/rate/channels) / iOS `AVAssetExportSession`.
- **Video**: Android **Media3 `Transformer` 1.10** via `AndroidVideoCompressor` (handles codec selection, HW/SW fallback, HDR tone-mapping, rotation — zero bundled codec binaries). iOS uses `AVAssetExportSession` / `AVAssetWriter`. See `kompressor/src/*/kotlin/co/crackn/kompressor/video/`.

## Build Commands

```bash
# Build everything
./gradlew build

# Fetch test fixtures (required for device + iOS tests; uses Git LFS)
./scripts/fetch-fixtures.sh

# Verify public API hasn't broken binary compatibility
./gradlew apiCheck

# Run all tests
./gradlew allTests

# Platform-specific tests
./gradlew iosSimulatorArm64Test
./gradlew testAndroidHostTest    # Android unit tests (run on host JVM)

# Run a single test class
./gradlew testAndroidHostTest --tests "co.crackn.kompressor.CompressionResultTest"

# Lint & format
./gradlew ktlintCheck            # Check formatting (ktlint 1.3.1)
./gradlew ktlintFormat           # Auto-fix formatting
./gradlew detekt                 # Static analysis (config: config/detekt/detekt.yml)

# Code coverage
./gradlew koverXmlReport         # Generate coverage report
./gradlew koverVerify            # Quality gate — minimum 85% coverage

# API reference (Dokka v2)
./gradlew :kompressor:dokkaHtml  # Alias for dokkaGeneratePublicationHtml; writes kompressor/build/dokka/html/
                                 # Also gated on every PR via the `dokka-build` job in pr.yml;
                                 # published to gh-pages + attached as the -javadoc.jar on every release.

# Publish to Maven Central (requires signing keys)
./gradlew publishToMavenCentral
```

## Architecture

### Module Structure

Two Gradle modules (see `settings.gradle.kts`):
- **`:kompressor`** — the published KMP library. All library source lives here.
- **`:sample`** — a sample app that exercises the library. Per user preference, it follows production-grade architecture (kotlin-inject DI, nav library, ViewModels, i18n) even though it's a sample.

The `iosApp/` directory at the repo root is a standalone Xcode project that consumes the KMP framework for iOS sample/testing — it is not a Gradle module.

### KMP Targets

- **Android** (API 24+) — `BitmapFactory` for images; Media3 `Transformer` for video and audio
- **iOS** (15+) — `UIImage`/Core Graphics for images; `AVAssetExportSession` / `AVAssetWriter` for video and audio

### Source Set Layout

```text
kompressor/src/
├── commonMain/              # Shared API: Kompressor interface, configs, result types
├── commonTest/              # Shared tests (Kotest + Turbine)
├── androidMain/             # Android implementations using MediaCodec, BitmapFactory
├── androidHostTest/         # Android unit tests (run on host JVM, not device)
├── androidDeviceTest/       # Android device-only tests — video golden + property tests live here (MediaCodec requires a real device/emulator)
├── iosMain/                 # iOS implementations using AVFoundation, VideoToolbox
└── iosTest/                 # iOS tests (run on simulator)
```

### Key API Pattern

Uses KMP `expect`/`actual` for the factory function:

```kotlin
// commonMain
expect fun createKompressor(): Kompressor

interface Kompressor {
    val image: ImageCompressor
    val video: VideoCompressor
    val audio: AudioCompressor

    suspend fun probe(inputPath: String): Result<SourceMediaInfo>
    fun canCompress(info: SourceMediaInfo): Supportability
}
```

Each platform provides an `actual fun createKompressor()` that returns platform-native implementations.

Use `probe` → `canCompress` to gate UX before attempting compression — returns a typed `Supportability` reflecting decoder/encoder availability on the running device (advisory, not a runtime guarantee).

All compress operations return `Result<CompressionResult>` — a unified result type with `inputSize`, `outputSize`, `compressionRatio`, and `durationMs`. Audio and video `compress()` methods accept an `onProgress: suspend (Float) -> Unit` callback for real-time progress tracking (0.0–1.0). Image compression has no progress callback because the underlying platform APIs are synchronous single-step operations.

### Important Patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cracknco/kompressor](https://github.com/cracknco/kompressor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
