---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

Swift Scribe is an AI-powered speech-to-text transcription application built exclusively for iOS 26/macOS 26+ using Apple's latest frameworks. It provides real-time voice transcription, on-device AI processing, speaker diarization, and intelligent note-taking with complete privacy protection.

## Essential Development Commands

### Standard Xcode Project Commands

**Building and Running:**
```bash
# Open project in Xcode
open SwiftScribe.xcodeproj

# Build from command line (requires Xcode)
xcodebuild -project SwiftScribe.xcodeproj -scheme SwiftScribe -destination 'platform=iOS Simulator,name=iPhone 15 Pro' build

# Build for macOS
xcodebuild -project SwiftScribe.xcodeproj -scheme SwiftScribe -destination 'platform=macOS' build

# Clean build folder
xcodebuild clean -project SwiftScribe.xcodeproj -scheme SwiftScribe
```

**Testing:**
```bash
# Run tests from command line
xcodebuild test -project SwiftScribe.xcodeproj -scheme SwiftScribe -destination 'platform=iOS Simulator,name=iPhone 15 Pro'

# Run tests for macOS
xcodebuild test -project SwiftScribe.xcodeproj -scheme SwiftScribe -destination 'platform=macOS'
```

**Swift Package Manager Integration:**
```bash
# Reset Swift Package Manager cache if dependencies have issues
rm -rf SwiftScribe.xcodeproj/project.xcworkspace/xcshareddata/swiftpm
# Then rebuild in Xcode to re-resolve packages
```

## Critical System Requirements

**IMPORTANT**: This project requires bleeding-edge Apple platforms:
- **iOS 26 Beta or newer** (will NOT work on iOS 25 or earlier)
- **macOS 26 Beta or newer** (will NOT work on macOS 25 or earlier)
- **Xcode Beta** with Swift 6.2+ toolchain
- **Apple Developer Account** with beta access

## High-Level Architecture

### Core Application Structure

**SwiftUI + SwiftData + Modern Concurrency Architecture:**
- Built entirely with SwiftUI for cross-platform UI
- SwiftData for object persistence (Core Data successor)
- Async/await and actors for concurrent operations
- Observable pattern using Swift 5.9+ `@Observable` macro

### Key Components Hierarchy

1. **App Layer** (`ScribeApp.swift`)
   - Main app entry point with SwiftData model container setup
   - Configures shared model context for memo persistence

2. **View Layer** (`Views/`)
   - `ContentView.swift`: Navigation split view (memo list + transcript detail)
   - `TranscriptView.swift`: Core recording interface with live transcription
   - `SettingsView.swift`: App configuration and preferences
   - Conditional compilation for iOS vs macOS UI differences

3. **Model Layer** (`Models/`)
   - `MemoModel.swift`: Core `Memo` class with SwiftData persistence, AI enhancement, and speaker attribution
   - `SpeakerModels.swift`: `Speaker` and `SpeakerSegment` models for diarization
   - `AppSettings.swift`: Observable settings for themes and diarization preferences

4. **Audio Processing** (`Audio/`)
   - `Recorder.swift`: Dual `AVAudioEngine` architecture (recording + playback engines)
   - `DiarizationManager.swift`: FluidAudio integration for real-time speaker identification

5. **Speech & AI** (`Transcription/` + `Helpers/`)
   - `Transcription.swift`: Apple Speech framework with async stream processing
   - `FoundationModelsHelper.swift`: On-device AI text generation using Apple's FoundationModels

### Apple Framework Dependencies

**Core Frameworks:**
- **SwiftUI**: Modern declarative UI framework
- **SwiftData**: Object persistence with `@Model` classes
- **Speech**: Real-time speech recognition with streaming
- **AVFoundation**: Audio recording, playback, and processing
- **FoundationModels**: On-device AI text generation (iOS 18+/macOS 15+)

**External Dependencies:**
- **FluidAudio**: Speaker diarization library for advanced speaker separation
  - Repository: `https://github.com/FluidInference/FluidAudio/`
  - Provides `DiarizerManager`, `DiarizationResult`, `TimedSpeakerSegment`

### Advanced Technical Features

1. **Dual Audio Engine Architecture**
   - Separate `AVAudioEngine` instances for recording and playback
   - Prevents conflicts and allows simultaneous recording/playback
   - Real-time audio processing with buffer management

2. **Rich Attribution System**
   - Custom `AttributedString` extensions for speaker identification
   - Color-coded text based on speaker identification
   - Confidence scoring and metadata embedding
   - Timeline-based character position mapping

3. **Real-Time Processing Pipeline**
   - Streaming transcription with live text updates
   - Optional concurrent speaker diarization
   - On-device AI enhancement for summaries and titles
   - Async actors for thread-safe audio processing

4. **Cross-Platform Considerations**
   - Conditional compilation using `#if os(iOS)` and `#if os(macOS)`
   - Platform-specific UI adaptations (navigation styles, toolbars)
   - Shared business logic with platform-specific presentation

## Development Guidelines

### Code Patterns and Conventions

**SwiftUI Observable Pattern:**
```swift
// Use @Observable classes for state management
@Observable
class AppSettings {
    var isDiarizationEnabled: Bool = false

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FluidInference/swift-scribe](https://github.com/FluidInference/swift-scribe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
