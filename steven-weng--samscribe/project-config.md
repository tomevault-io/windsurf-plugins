---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SamScribe is an open-source macOS transcription application that captures and transcribes audio from both the microphone and running applications (e.g., Zoom, Chrome, Teams) in real-time using FluidAudio for ASR (Automatic Speech Recognition) and speaker diarization. Features cross-recording speaker recognition using persistent voice embeddings.

## Build Commands

```bash
# Build the project
xcodebuild -project SamScribe.xcodeproj -scheme SamScribe -configuration Debug

# Build for release
xcodebuild -project SamScribe.xcodeproj -scheme SamScribe -configuration Release

# Clean build
xcodebuild clean -project SamScribe.xcodeproj -scheme SamScribe

# Open in Xcode
open SamScribe.xcodeproj
```

## Key Architecture

### Audio Pipeline Flow

1. **AudioManager** (MainActor): Orchestrates the entire audio capture and transcription pipeline
   - Manages microphone input via `AudioInput`
   - Manages per-process application audio via `ApplicationAudio`
   - Monitors audio processes via `AudioMonitor`
   - Routes audio buffers to `Transcriber`

2. **AudioInput** (Actor): Captures microphone audio using AVAudioEngine
   - Converts audio to 16kHz mono Float32 (FluidAudio format)
   - Implements mute control
   - Handles device assignment with retry logic

3. **ApplicationAudio** (Actor): Captures per-process audio using ScreenCaptureKit
   - Creates individual SCStream per audio process
   - Converts 48kHz stereo to 16kHz mono
   - Manages stream lifecycle with grace periods

4. **AudioMonitor** (@MainActor): Monitors system audio processes
   - Polls CoreAudio every 2 seconds for active audio processes
   - Implements 3-second start delay (prevents short audio spikes)
   - Implements 5-second grace period (prevents stream churn)
   - Filters for audio apps only (browsers, meeting apps, etc.)

5. **Transcriber** (Actor): Performs ASR and diarization using FluidAudio
   - Accumulates 10-second audio chunks per source
   - Processes chunks independently for each audio source (microphone vs app audio)
   - Performs ASR transcription with confidence scores
   - Performs speaker diarization and extracts speaker embeddings (256-dimensional)
   - Configured with tuned parameters: clusteringThreshold: 0.5, minSpeechDuration: 0.5s, minSilenceGap: 0.2s

6. **SpeakerManager** (@MainActor): Manages global speaker recognition
   - Matches voice embeddings using Accelerate-optimized cosine similarity
   - Uses FluidAudio's AssignmentConfig.macOS (0.65 distance threshold)
   - Creates and persists Speaker entities in SwiftData
   - Enables cross-recording speaker identification

### Data Flow

```
AudioInput (mic) ──┐
                   ├──> Transcriber ──> TranscriptionResult ──┐
ApplicationAudio ──┘    (10s chunks)    (with embeddings)    │
(per-process)                                                 │
                                                              ▼
                                                   TranscriptionsStore
                                                              │
                                                              ├──> SpeakerManager (match/create Speaker)
                                                              │
                                                              ▼
                                                          SwiftData
                                                (Recording, TranscriptionSegment, Speaker)
```

### Audio Format Conversions

- **Input formats**: Variable (44.1kHz-48kHz, mono/stereo)
- **Target format**: 16kHz mono Float32 (FluidAudio requirement)
- **Conversion**: AudioConverter (resampler using vDSP/Accelerate)
- **Chunk size**: 10 seconds = 160,000 samples @ 16kHz

### Permission Requirements

1. **Microphone**: Required for AudioInput (NSMicrophoneUsageDescription)
   - Automatically requested on app launch if not determined
   - Uses AVCaptureDevice.requestAccess(for: .audio)

2. **Screen Recording**: Required for ScreenCaptureKit to capture app audio (NSScreenCaptureUsageDescription)
   - Automatically requested when user starts recording
   - Uses CGPreflightScreenCaptureAccess() and CGRequestScreenCaptureAccess()
   - Alert guides user to System Settings if denied

3. **Permission Flow**: Managed in TranscriptionView
   - Checks both permissions on app launch
   - Button disabled until permissions checked
   - Shows appropriate alerts with "Open Settings" option

4. **Sandbox entitlements**: See SamScribe.entitlements

### Process Lifecycle Management

- **3-second start delay**: Waits 3s after detecting audio before creating SCStream (prevents spurious stream creation)
- **5-second grace period**: Waits 5s after process stops before destroying SCStream (handles temporary audio pauses)
- **Helper process mapping**: Chrome Helper → Chrome, Safari Renderer → Safari, etc.

### Key Files by Function

**Audio Capture**:
- `AudioInput.swift`: Microphone capture with AVAudioEngine
- `ApplicationAudio.swift`: Per-process app audio with ScreenCaptureKit
- `AudioMonitor.swift`: System audio process monitoring

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Steven-Weng/SamScribe](https://github.com/Steven-Weng/SamScribe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
