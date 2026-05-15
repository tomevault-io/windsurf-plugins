---
trigger: always_on
description: VERSION                    # App version (read by build scripts)
---

# Meeting Transcriber

## Project Structure

```
VERSION                    # App version (read by build scripts)
app/MeetingTranscriber/    # Swift macOS menu bar app (SPM)
  Package.swift            # SPM manifest (WhisperKit + FluidAudio + AudioTapLib runtime deps; ViewInspector + SnapshotTesting test deps)
  Sources/
    MeetingTranscriberApp.swift  # @main, UI shell (scenes, NSOpenPanel, NSWorkspace)
    AppState.swift         # @Observable @MainActor ViewModel (business state, badge logic, pipeline wiring)
    AudioConstants.swift   # Shared audio pipeline constants (target sample rate)
    MenuBarView.swift      # Menu bar dropdown UI
    MenuBarIcon.swift      # Animated waveform menu bar icon + BadgeKind.compute() pure function
    SettingsView.swift     # Settings window (TabView shell hosting six sub-views in Settings/)
    Settings/
      GeneralSettingsView.swift  # Apps to Watch · Detection · Updates
      AudioSettingsView.swift    # Microphone device · VAD settings
      TranscriptionSettingsView.swift  # ASR engine picker + per-engine options
      SpeakersSettingsView.swift # Diarization · Mic Speaker Name · Known Voices · Recognition Stats
      OutputSettingsView.swift   # LLM provider · protocol language · output folder · prompt
      AdvancedSettingsView.swift # Permissions · Diagnostics · About
      View+RecordOnly.swift      # `recordOnlyDisabled(_:)` SwiftUI modifier (dim + disable downstream sections)
    SpeakerNamingView.swift # Speaker naming dialog + AccessibleTextField
    KnownVoicesView.swift  # Speaker DB management UI (rename, delete, merge entries)
    RecognitionStatsView.swift # Recognition stats display (aggregate counts from recognition_log.jsonl)
    VoiceEnrollmentView.swift  # Voice enrollment sheet (seed speakers.json from audio file)
    AppPickerView.swift    # App picker for manual recording
    AppPaths.swift         # Centralized paths (ipcDir, dataDir, logSubsystem, speakersDB)
    AppSettings.swift      # @Observable settings (UserDefaults + file-based secrets)
    AXHelper.swift         # Shared accessibility API helper
    NotificationManager.swift # macOS notifications
    KeychainHelper.swift   # Keychain CRUD (legacy/test-only, token now file-based)
    TranscriberStatus.swift # Status + MeetingInfo models
    TranscribingEngine.swift # TranscribingEngine protocol + mergeDualSourceSegments default impl
    WhisperKitEngine.swift # WhisperKit transcription engine (CoreML/ANE, 99+ languages)
    ParakeetEngine.swift   # NVIDIA Parakeet TDT v3 engine via FluidAudio (CoreML/ANE, 25 EU languages)
    Qwen3AsrEngine.swift   # Qwen3-ASR 0.6B engine via FluidAudio (CoreML/ANE, 30 languages, macOS 15+)
    FluidDiarizer.swift    # CoreML-based speaker diarization via FluidAudio (on-device, OfflineDiarizer + Sortformer modes)
    FluidVAD.swift         # VAD preprocessing via FluidAudio Silero v6 (silence trimming + timeline remapping)
    SpeakerMatcher.swift   # Speaker embedding DB + cosine similarity matching
    RecognitionStats.swift # Recognition event logging + aggregate stats model (recognition_log.jsonl)
    DiarizationProcess.swift  # DiarizationProvider protocol + result types
    PipelineQueue.swift    # Decouples recording from post-processing (transcription → diarization → protocol)
    PipelineJob.swift      # Pipeline job model
    ProtocolGenerator.swift   # Shared protocol utilities: prompts, file I/O, ProtocolError
    ClaudeCLIProtocolGenerator.swift # Claude CLI subprocess protocol generation (#if !APPSTORE)
    OpenAIProtocolGenerator.swift # OpenAI-compatible API protocol generation (Ollama, LM Studio, etc.)
    WatchLoop.swift        # @MainActor watch loop: detect → record → enqueue PipelineJob
    DualSourceRecorder.swift  # App audio (AudioTapLib) + mic recording (captures startTime in start())
    MeetingDetecting.swift # MeetingDetecting protocol + DetectedMeeting model
    MeetingDetector.swift  # Window title matching (counts each pattern once per poll)
    FFmpegHelper.swift     # ffmpeg CLI detection + audio extraction for MKV/WebM/OGG
    AudioMixer.swift       # Multi-format audio loading (WAV/MP3/M4A/MP4 via AVAsset fallback, MKV/WebM/OGG via ffmpeg) + mixing to 16kHz mono
    MicRecorder.swift      # Microphone recording via AVAudioEngine
    PermissionHealthCheck.swift # Permission health check (TCC verdict + live probe → PermissionStatus)
    PermissionRow.swift    # Permission status row UI component
    Permissions.swift      # Permission checks (mic, screen recording)
    ParticipantReader.swift # Reads meeting participants via accessibility
    MeetingPatterns.swift  # App-specific window title patterns
    PowerAssertionDetector.swift  # Meeting detection via IOKit power assertions (sandbox-safe)
    UpdateChecker.swift    # GitHub release update checker
    DebugRPCServer.swift   # Localhost HTTP RPC for shell-driven inspection (#if !APPSTORE, env-gated by MEETINGTRANSCRIBER_DEBUG_RPC=1)
    Assets.xcassets        # App icon assets
    Info.plist             # Bundle metadata
  Entitlements/
    Homebrew.entitlements  # Mic only (Homebrew/direct distribution)
    AppStore.entitlements  # Sandbox + mic + network + file picker (App Store)
  Tests/                   # Swift tests (XCTest + ViewInspector)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pasrom/meeting-transcriber](https://github.com/pasrom/meeting-transcriber) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
