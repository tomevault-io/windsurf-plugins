---
trigger: always_on
description: Scope: Entire repository
---

# Repository Guidelines

Scope: Entire repository  
Owner: HermesWhisper Development Team
Last updated: May 9, 2026

Note to agents and contributors: Keep this document up to date with any changes.

## Project Structure & Module Organization
HermesWhisper stores SwiftUI sources under `HermesWhisper/`, with views, view models, and helpers grouped by feature. Shared assets live in `HermesWhisper/Assets.xcassets`, while project settings and entitlements sit beside the sources. Unit targets reside in `HermesWhisperTests/`, and UI automation lives in `HermesWhisperUITests/`. Local build artifacts accumulate under `build/`, and Xcode writes derived data to `DerivedData_WW/`.

### Architecture Overview
Core components: `DictationViewModel` (orchestrates recording → transcription → LLM → insertion), `HistoryStore` & `ConversationHistoryStore` (file-based JSON persistence), provider protocols (`TranscriptionProvider`, `LLMProvider`), and service layers (`AudioRecorder`, `ScreenContextService`, `InsertionService`, `HotkeyManager`). Storage paths: `~/Library/Application Support/HermesWhisper/` for history entries, audio files, screen captures, and conversation state. On first launch after the rename, local app support data is copied from the legacy `WonderWhisper` directory if needed. API keys stored in macOS Keychain via `KeychainService`.

### Microphone Selection
The app includes a persistent microphone selection feature accessible from the sidebar. Users can choose between system default (auto-switches with device changes) or override with a specific microphone. Selection is persisted via `AudioInputSelection` in `AudioDeviceManager.swift` and displayed in `MicrophoneSelectionView.swift`.

## Feature Scope & Providers
- The app ships a single window with eight sidebar tabs: Hermes, History, Dictation, Command, Vocabulary, Microphone, Permissions, and Settings. Scratchpad, Pro mode, and file transcription workflows have been removed; keep new work within these surfaces.
- Transcription uses Groq Whisper Large V3 Turbo (`groq-streaming`), local Parakeet V3 (`parakeet-local`), Soniox V4 (`soniox-streaming`), OpenRouter speech-to-text models (`openrouter-transcription`), or xAI Grok Speech-to-Text (`xai-stt`). Users pick the engine in **Settings → Transcription engine**; default is Parakeet. Do not reintroduce other providers without explicitly updating this document.
- All LLM requests route through OpenRouter. Additional providers (Groq Chat, Cerebras, Ollama, etc.) are no longer part of the shipping build, so any new integration must be justified and added here.

## Build, Test, and Development Commands
Use `open "HermesWhisper.xcodeproj"` to launch Xcode. For a CLI build, run `xcodebuild -project "HermesWhisper.xcodeproj" -scheme "HermesWhisper" -configuration Debug build`. Execute tests with `xcodebuild -project "HermesWhisper.xcodeproj" -scheme "HermesWhisper" -destination 'platform=macOS' test`. To run a single test, use `xcodebuild -project "HermesWhisper.xcodeproj" -scheme "HermesWhisper" -destination 'platform=macOS' test -only-testing:HermesWhisperTests/HermesWhisperTests/testName`. After a successful script build, `open build/Build/Products/Debug/HermesWhisper.app` launches the latest artifact. The project uses Swift Testing framework (not XCTest) with `@Test` annotations.

## Coding Style & Naming Conventions
Adopt 2-space indentation and keep lines near 100 characters. Name types with PascalCase, functions and variables with camelCase, and prefer `static let` for constants. Match filenames to the primary type (`AudioTranscriber.swift`). Imports should be organized: Foundation first, then Apple frameworks (SwiftUI, AVFoundation, etc.), then `@testable import` in tests. Favor small SwiftUI views, avoid force unwraps (use `guard` or optional chaining), use explicit error handling with `do-catch` or `throws`, and add SwiftUI previews when practical. One primary type per file. Run `swiftformat .` and `swiftlint` before posting changes when tooling is available.

## Testing Guidelines
Tests use Swift Testing framework (not XCTest). Use `@Test` annotation and name functions descriptively: `func audioPreprocessorProducesNormalized16BitOutput()` or `func http2SessionsExposePreferredConfiguration()`. Use `#expect` for assertions, `.disabled("reason")` to skip tests. Target audio, transcription, and provider logic first, then UI flows. Aim for ≥80% coverage on critical modules. Run `xcodebuild ... test` or Xcode's Test action prior to opening a pull request.

## Commit & Pull Request Guidelines
Write imperative, focused commits such as `fix: handle microphone permission denial`. In PRs, describe the approach, link related issues, and flag risks. Include screenshots or GIFs for UI updates and confirm build, tests, and lint all succeed locally. Note any gaps explicitly.

## Security & Configuration Tips
Never commit secrets; use local `.xcconfig` files or Keychain values instead. Review entitlements and Hardened Runtime settings when adding capabilities. Avoid private macOS APIs and audit third-party dependencies periodically.

## Documentation Index

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dkapo88/hermeswhisper](https://github.com/dkapo88/hermeswhisper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
