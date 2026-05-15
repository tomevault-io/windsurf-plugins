---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

OpenSuperWhisper is a macOS application that provides real-time audio transcription using the Whisper model. Built with SwiftUI and native macOS APIs, it offers seamless audio recording and transcription with customizable settings and global keyboard shortcuts.

**Platform Requirements**: macOS 14.0+ (Sonoma), ARM64 (Apple Silicon) only

## Development Commands

### Building the Project

**Initial Setup**:
```bash
git submodule update --init --recursive
brew install cmake
gem install xcpretty  # Optional: for prettier build output
```

**Development Build**:
```bash
./run.sh build    # Build only
./run.sh          # Build and run
```

**Manual Build Process**:
```bash
# Configure libwhisper (native C++ library)
cmake -G Xcode -B libwhisper/build -S libwhisper

# Build via Xcode command line
xcodebuild -scheme OpenSuperWhisper -configuration Debug \
  -destination 'platform=macOS,arch=arm64' \
  -derivedDataPath build build
```

### Testing

**Run All Tests**:
```bash
xcodebuild test -scheme OpenSuperWhisper -destination 'platform=macOS,arch=arm64'
```

**Individual Test Targets**:
```bash
xcodebuild test -scheme OpenSuperWhisper -only-testing:OpenSuperWhisperTests
xcodebuild test -scheme OpenSuperWhisper -only-testing:OpenSuperWhisperUITests
```

**Run Specific Test Files**:
```bash
# STT-related tests
xcodebuild test -scheme OpenSuperWhisper -only-testing:OpenSuperWhisperTests/STTProviderFactoryTests
xcodebuild test -scheme OpenSuperWhisper -only-testing:OpenSuperWhisperTests/MistralVoxtralProviderTests
xcodebuild test -scheme OpenSuperWhisper -only-testing:OpenSuperWhisperTests/EnhancedTranscriptionServiceTests

# Text improvement tests
xcodebuild test -scheme OpenSuperWhisper -only-testing:OpenSuperWhisperTests/TextImprovementServiceTests
xcodebuild test -scheme OpenSuperWhisper -only-testing:OpenSuperWhisperTests/TextImprovementConfigurationTests
```

### Release Build

**Signed Release Build**:
```bash
./notarize_app.sh "Developer ID Application: Your Name (TEAM_ID)"
```

**Full Release Pipeline**:
```bash
./make_release.sh 0.0.4 "Developer ID Application: Your Name (TEAM_ID)" ghp_xxxxx
```

## Architecture Overview

### Core Components

**App Entry Point**: `OpenSuperWhisperApp.swift`
- SwiftUI app with `@main` attribute
- Singleton `AppState` for global state management
- Menu bar app with system tray integration
- Conditional onboarding flow

**Audio Pipeline**: `AudioRecorder.swift`
- Singleton pattern for thread-safe recording
- AVFoundation integration with optimized settings (16kHz, mono, 16-bit PCM)
- Dynamic audio device monitoring
- Temporary file management with automatic cleanup

**Transcription Engine**: `TranscriptionService.swift` & `EnhancedTranscriptionService.swift`
- MainActor isolation for UI updates
- Asynchronous processing with cancellation support
- Audio conversion pipeline (44.1kHz → 16kHz mono float32)
- Real-time progress tracking via segment callbacks
- Multi-provider STT support (Local Whisper, Mistral Voxtral cloud API)
- Provider factory pattern for extensible STT backends

**Model Management**: `WhisperModelManager.swift`
- Application Support directory for model storage
- Default model bundling (`ggml-tiny.en.bin`)
- Progress-tracked downloads with resume capability

**Swift Wrapper**: `Whis/Whis.swift`
- Memory-safe C interop with `OpaquePointer` management
- Context and state management for thread safety
- Real-time callback integration
- OpenVINO hardware acceleration support

**STT Provider System**: `STT/` directory
- `STTProvider.swift`: Core protocol for all STT implementations
- `MistralVoxtralProvider.swift`: Cloud-based transcription via Mistral AI API
- `STTProviderFactory.swift`: Factory pattern for provider instantiation
- `SecureStorage.swift`: Keychain management for API credentials
- `STTConfigurations.swift`: Provider-specific configuration management

**Text Improvement Service**: `TextImprovementService.swift`
- LLM-powered text enhancement via cloud APIs
- Configurable provider support (OpenAI, Anthropic, etc.)
- Real-time progress tracking and error handling
- Secure credential management via Keychain

### Key Architectural Patterns

- **MVVM**: Clear separation between view models and business logic
- **Observer Pattern**: `@Published` properties for reactive UI updates
- **Singleton Pattern**: Shared managers for core services
- **Delegation Pattern**: Audio recorder and URL session delegates
- **Async/Await**: Modern concurrency for audio processing

### Directory Structure

```
OpenSuperWhisper/
├── OpenSuperWhisperApp.swift     # Main app entry point
├── AudioRecorder.swift           # Audio recording pipeline
├── TranscriptionService.swift    # Legacy transcription service
├── EnhancedTranscriptionService.swift # New unified transcription service
├── WhisperModelManager.swift     # ML model lifecycle
├── Settings.swift                # Configuration management
├── ShortcutManager.swift         # Global keyboard shortcuts
├── TextImprovementService.swift  # LLM text enhancement
├── TextImprovementConfiguration.swift # Text improvement settings

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Schreezer/STT](https://github.com/Schreezer/STT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
