---
trigger: always_on
description: Scope: Entire repository
---

# Repository Guidelines

Scope: Entire repository  
Owner: WonderWhisper Development Team
Last updated: July 16, 2026

Note to agents and contributors: Keep this document up to date with any changes.

## Project Structure & Module Organization
WonderWhisper stores SwiftUI sources under `WonderWhisper/`, with views, view models, and helpers grouped by feature. Shared assets live in `WonderWhisper/Assets.xcassets`, while project settings and entitlements sit beside the sources. Unit tests reside in `WonderWhisperTests/`. Local build artifacts accumulate under `build/`, and Xcode writes derived data to `DerivedData_WW/`.

### Architecture Overview
Core components: `DictationViewModel` (orchestrates recording → transcription → OpenRouter → insertion), `MeetingCoordinator` (orchestrates dual-source meeting capture, streaming transcription, notes, and export), `HistoryStore` & `ConversationHistoryStore` (file-based JSON persistence), the `TranscriptionProvider` protocol, and service layers (`AudioRecorder`, `ScreenContextService`, `InsertionService`, `PromptHotkeyManager`). `HotkeyManager` retains only the Paste Last Carbon shortcut and shared shortcut value types. Storage paths remain under `~/Library/Application Support/HermesWhisper/` for compatibility with existing history, meeting audio, screenshots, and conversation state. The bundle identifier and Keychain service likewise retain their Hermes-era values so the WonderWhisper rebrand does not reset macOS permissions, settings, or credentials. API keys are stored in macOS Keychain via `KeychainService`.

### Microphone Selection
The app includes persistent microphone priority ordering in the sidebar. Users can choose the system
default or rank remembered microphones; unavailable choices fall through to the highest-priority
connected device and then the system default. Selection is persisted via `AudioInputSelection` and
`AudioDeviceManager` and displayed in `MicrophoneSelectionView.swift`.

## Feature Scope & Providers
- The app ships a single window with eleven sidebar tabs: History, Dictation, Command, Meetings,
  Beeper, Hermes, Vocabulary, Microphone, Compare, Permissions, and Settings. Scratchpad, Pro mode,
  and file transcription workflows have been removed; keep new work within these surfaces.
- Transcription uses Groq Whisper Large V3 Turbo through stable file upload (legacy engine ID `groq-streaming`), local Parakeet (`parakeet-local`), Soniox V5 (`soniox-streaming`), OpenRouter speech-to-text models (`openrouter-transcription`), or xAI Grok Speech-to-Text (`xai-stt`). Users pick the engine in **Settings → Transcription engine**; default is Parakeet. Do not reintroduce other providers without explicitly updating this document.
- Meetings retain separate microphone and system-audio capture tracks. System audio comes from a
  private Core Audio process tap before output volume and device routing, while ScreenCaptureKit
  supplies the selected microphone. Parakeet Unified remains the free on-device default with
  source-specific inference. Soniox V5 is an opt-in cloud beta whose default mode normalizes
  variable capture callbacks into fixed 100 ms source frames,
  timestamp-aligns both sources, uses Accelerate-backed adaptive system-audio echo reduction,
  and sends one mixed WebSocket with speaker metadata for approximately $0.12 per meeting hour. A
  two-WebSocket source-separated mode remains available as a fallback at approximately $0.24 per hour. Audio
  is retained as bounded one-minute CAF segments under `Meetings/<uuid>/`. Manual sessions capture
  all Mac system audio; automatically detected sessions restrict capture to the detected
  application scope. Trigger apps are editable: Slack and supported browsers retain strict
  Huddle/Google Meet evidence, while explicitly configured standalone apps may start on microphone
  use. Core Audio process activity wakes strict automatic-start validation immediately, with a
  two-second fallback heartbeat, while an active meeting tolerates browser
  title and individual audio-signal dropouts before a 30-second confirmed stop. Soniox non-final
  text is transient UI only; Stop ends local capture immediately while final tokens, notes, and
  export finish in a session-scoped background task. Failed live-stream tails are recovered from
  retained CAF segments with local Parakeet Unified. Full Soniox failures first recover both retained
  raw tracks with Soniox Async V5 so system-audio speaker labels survive; if that fails, local Parakeet
  remains the fallback. One cumulative late 100 ms mixed frame is tolerated before full recovery.
  Mixed capture runs on a dedicated serial ingestion worker so Soniox
  token and UI callbacks cannot starve audio delivery. If live ingestion falls behind, transcription pauses and is recovered
  later while durable audio capture continues uninterrupted. The companion includes a durable Manual notes
  tab backed by an atomically saved local sidecar; those notes remain separate from generated Markdown,
  appear in history and exports, and join the transcript only when cloud-generated notes are opted in.
  Generated notes are cloud opt-in. Optional live
  context uses its own fast OpenRouter model and sends a bounded recent transcript window to extract useful

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dkapo88/WonderWhisper-macOS](https://github.com/dkapo88/WonderWhisper-macOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
