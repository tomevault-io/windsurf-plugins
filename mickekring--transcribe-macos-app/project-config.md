---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

This is a native macOS SwiftUI app. Open and build with Xcode:

```bash
open Transcribe.xcodeproj
# Build: Cmd+B | Run: Cmd+R
```

The project requires macOS 26+ (Tahoe) and runs on Apple Silicon (M1+).

### Known Xcode 26 Build Issue

Building from Xcode's UI fails with `___llvm_profile_runtime` undefined symbol (yyjson, a pure C SPM transitive dependency of WhisperKit). This is an Xcode 26 bug where `CLANG_COVERAGE_MAPPING` defaults to YES and pure C SPM package targets get coverage instrumentation but miss `-fprofile-instr-generate` in their linker invocation. Build from the command line instead:

```bash
xcodebuild build -scheme Transcribe CLANG_COVERAGE_MAPPING=NO
```

**Important**: Always rebuild after making code changes to verify they compile and so the user can test them. Never skip the build step.

## Architecture

### Entry Point
- `Transcribe/TranscribeApp.swift` - Main app with SwiftUI lifecycle
- `Transcribe/AppDelegate.swift` - NSApplicationDelegate for status bar menu, file cleanup, window management

### Core Services

**Transcription Pipeline:**
- `WhisperKitService.swift` - Primary transcription engine using WhisperKit (CoreML). Handles model loading, transcription with progress streaming, and segment generation.
- `UnifiedTranscriptionService.swift` - Routes between local (WhisperKit) and cloud (Berget) services. Manages model state and active service selection.
- `BergetTranscriptionService.swift` - Cloud transcription via Berget AI API (requires API key stored in Keychain)
- `TranscriptionService.swift` - Lightweight streaming wrapper around WhisperKitService

**Audio Processing:**
- `Services/AudioPreprocessor.swift` - Audio format conversion and preprocessing. Automatically extracts audio from video containers (`.mp4`, `.mov`, etc.) and converts non-native formats to 16kHz mono WAV using `AVAssetReader`/`AVAssetWriter`. Used by both local (WhisperKit) and cloud (Berget) transcription paths.

**System Audio Capture:**
- `Services/SystemAudioCaptureService.swift` - Captures system audio output using ScreenCaptureKit with optional microphone mixing. Two-phase architecture: monitoring (level meter, permission prompt) and recording (48kHz stereo float32 WAV). Supports mixing mic input via `AVAudioEngine` + `AudioRingBuffer` for meeting recording (both sides of a conversation). Includes CoreAudio device enumeration and selection for mic input.
- `Services/AudioRingBuffer.swift` - Thread-safe SPSC ring buffer using `os_unfair_lock`. Decouples mic capture (AVAudioEngine thread) from system audio capture (SCStream callback thread) during mixed recording.

**LLM Integration:**
- `Services/LLMService.swift` - Text processing via Berget AI or Ollama LLM APIs (summarization, action points, etc.)

**Model Support:**
- KB Whisper models (Swedish-optimized): `kb_whisper-base/small/medium/large-coreml`
- OpenAI Whisper models: `openai_whisper-base/small/medium/large-v2/large-v3`
- Cloud: Berget KB Whisper Large (requires API key)

KB Whisper models load from `mickekringai/kb-whisper-coreml` HuggingFace repo. Default model (`kb_whisper-small-coreml`) auto-downloads on first launch.

### Key Managers
- `ModelManager.swift` - Model downloads, availability checking, and storage management
- `LanguageManager.swift` - Language selection (100+ languages)
- `SimplifiedManagers.swift` - Contains `SettingsManager` (app preferences, API keys, text processing prompts) and `KeychainHelper` (Keychain wrapper for API key storage)
- `LocalizationManager.swift` - UI localization (English/Swedish) via `localized(_:)` helper

### Views
- `ContentView.swift` - Main window with toolbar (file picker, recording, YouTube, settings)
- `TranscriptionView.swift` - Transcription display with audio player, export, and text processing
- `RecordingView.swift` - Built-in audio recording with live level meter, device selector, and playback
- `SystemAudioRecordingView.swift` - System audio capture with live level meter, record/stop, playback, and transcription
- `YouTubeTranscriptionView.swift` - YouTube URL transcription workflow
- `SettingsView.swift` - Preferences with model management, text processing prompts, LLM settings

### Data Models
- `TranscriptionModels.swift` - `TranscriptionResult`, `TranscriptionSegment`, `TextProcessingPrompt`, `AudioInputDevice`
- `AppState.swift` - Navigation state (`currentTranscriptionURL`, `showTranscriptionView`, `showRecordingView`, `showSystemAudioView`)

### File Storage & Security
All temporary files auto-cleanup on app quit and on startup (handles force-quit scenarios):
- Recordings: `~/Library/Caches/Transcribe/Recordings/`
- YouTube downloads: `~/Library/Caches/Transcribe/YouTube/`
- Cleanup handled in `AppDelegate.cleanupTemporaryFiles()`

API keys are stored in macOS Keychain via `KeychainHelper` (not UserDefaults). Legacy UserDefaults keys are migrated to Keychain on launch. No `print()` statements in production code — all debug logging has been removed.

## Key Patterns

### State Management
Uses `@EnvironmentObject` for global state:
- `AppState` - Navigation state (current file, view visibility)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mickekring/Transcribe-MacOS-App](https://github.com/mickekring/Transcribe-MacOS-App) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
