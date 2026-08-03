---
trigger: always_on
description: macOS app (macOS 26+ / Tahoe): AI-powered meeting notes + document editing. Privacy-first — all AI inference, transcription, and data processing runs locally on-device via MLX. By default nothing leaves the laptop; the only network calls are on-device model downloads, Sparkle update checks, and opt-in features the user explicitly enables (web search, external AI providers).
---

# Logue — Development Guidelines

## Project Overview

macOS app (macOS 26+ / Tahoe): AI-powered meeting notes + document editing. Privacy-first — all AI inference, transcription, and data processing runs locally on-device via MLX. By default nothing leaves the laptop; the only network calls are on-device model downloads, Sparkle update checks, and opt-in features the user explicitly enables (web search, external AI providers).

- **Bundle ID:** `com.bitwize.logue`
- **Source:** `Logue/`, **Tests:** `LogueTests/`
- **Swift 5.9 + SwiftUI + AppKit**
- **Build system:** XcodeGen — run `xcodegen generate` after adding new `.swift` files or changing `project.yml`
- **Build:** `xcodebuild build -project Logue.xcodeproj -scheme Logue -destination 'platform=macOS'`
- **MLX prerequisite:** `xcodebuild -downloadComponent MetalToolchain`
- **Test (LLM integration):** `xcodebuild test -project Logue.xcodeproj -scheme Logue -destination 'platform=macOS' -only-testing:LogueTests/<SuiteName>`

## Dependencies

| Dependency | Source | Purpose |
| ---------- | ------ | ------- |
| `mlx-swift-lm` (`MLXLLM`, `MLXLMCommon`) | remote SPM | On-device MLX LLM inference on Apple Silicon |
| `swift-transformers-mlx` (`MLXLMTransformers`) | remote SPM | Tokenizers / model plumbing for MLX |
| `swift-hf-api-mlx` (`MLXLMHFAPI`) | remote SPM | Hugging Face model download / hub cache |
| `LangGraph` | `Vendor/LangGraph-Swift` (submodule) | Agent graph framework (used by `WritingAgentGraph`) |
| `Markdown` | `Vendor/swift-markdown` (submodule) | Markdown parsing |
| `FluidAudio` | remote SPM | Speaker diarization — streaming Sortformer + batch fallback |
| `Textual` | remote SPM | Rich-text rendering |
| `Sparkle` | remote SPM | In-app auto-update (GitHub-hosted appcast) |
| Apple `Speech` framework | system SDK | `SpeechTranscriber` for real-time transcription (macOS 26+) |

## Architecture

### Audio Pipeline (direct streaming — no chunking, no backpressure queue)

- `AudioRecorder` — mic capture, raw `AVAudioPCMBuffer` via callback
- `SystemAudioCapture` — ScreenCaptureKit system audio, `CMSampleBuffer` → `AVAudioPCMBuffer`
- `BufferConverter` — `AVAudioConverter` wrapper for format conversion
- `SpeechTranscriberEngine` — streams raw audio → `SpeechAnalyzer` → `TranscriptSegment`
- `RecordingSessionManager` — orchestrates engines + diarization; uses `RecordingState` enum (`.idle`, `.starting`, `.recording`, `.stopping`)
- `DiarizationManager` — FluidAudio wrapper; streaming Sortformer (primary) + batch accumulation fallback

### LLM Engine

- `LLMEngine` (actor) — centralized inference, serialized via `inferenceGate`. Core: `complete(system:prompt:)`. Convenience: `generate()`, `chat()`, `analyzeRaw()`, `rephrase()` (in extension files). Also hosts LangGraph writing-agent nodes and streaming analysis. All extension methods MUST route through `complete()`/`completeStream()`.
- `LLMEngineStatus` (@MainActor @Observable) — singleton busy flag (`isBusy`) driven by `LLMEngine.inferenceQueueDepth`. UI views use `.disabled(LLMEngineStatus.shared.isBusy)` to prevent concurrent AI operations.
- `ModelManager` (@MainActor @Observable) — model downloads, activation, endpoint scanning (split: +Download, +Discovery, +HuggingFace)

### Data Layer

- `MeetingStore` (@MainActor @Observable) — encrypted JSON persistence (split: +AI, +Diarization, +Metadata, +Persistence, +Search, +SeedData, +WelcomeMeeting, Protocols)
- `MeetingNote` (struct, Codable, Sendable) — segments, speakers, speakerSegments, hasSpeakerData
- `EncryptionManager` — AES-256-GCM at rest, 7-day migration window for legacy unencrypted data

### Tests (Swift Testing framework — @Suite, @Test, #expect, NOT XCTest)

- 94 `@Test` methods across 9 files in `LogueTests/LLMIntegration/`
- `LLMTestHarness.swift` — shared harness with `LenientSuggestionItem`, `repairTruncatedJSON()`, `stripMarkdownFences()`
- Tests run real inference against local MLX model
- Grammar suite uses 10-minute timeout; all others 5 minutes

## Code Standards (Enforced by Review)

### Security

- **Never use `[0]` on FileManager URL arrays.** Use `.first ?? URL.temporaryDirectory` — the array can be empty on edge-case system configurations.
- **Always wrap user content in XML delimiters** when injecting into LLM prompts:
  - Meeting transcripts → `<transcript>...</transcript>`
  - Document content → `<content>...</content>`
  - PII categories → `<categories>...</categories>`
  - This applies to ALL prompt construction — summaries, titles, chat, search, fact-check, vocabulary, PII detection, space suggestions. No exceptions.
- **Require HTTPS for all user-supplied endpoints** (except localhost/127.0.0.1). Validate before saving.
- **Sanitize all user-provided strings** (titles, keywords, model names, space names) before embedding in LLM prompts — truncate length, strip control characters. Use the pattern: `String($0.prefix(N)).filter { !$0.isNewline && $0.asciiValue != 0 }`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bitwize-ai/Logue](https://github.com/bitwize-ai/Logue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
