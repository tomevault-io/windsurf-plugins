---
trigger: always_on
description: - VibeScribe is a macOS 15+ SwiftUI app that records microphone and optional system audio, transcribes it with Whisper-compatible servers, and generates AI summaries through OpenAI-style chat APIs.
---

# VibeScribe Agent Handbook

## App Overview
- VibeScribe is a macOS 15+ SwiftUI app that records microphone and optional system audio, transcribes it with Whisper-compatible servers, and generates AI summaries through OpenAI-style chat APIs.  
- Status bar integration plus a floating recording overlay keeps the workflow lightweight while SwiftData persists recordings, settings, and generated content.

## Code Layout
- `VibeScribe/Views/` – SwiftUI scenes and components (sidebar, detail view, recording overlay, settings).  
- `VibeScribe/Managers/` – Recording, playback, import, overlay window, transcription, summarization, and waveform caching services.  
- `VibeScribe/Models/` – SwiftData models (`Record`, `AppSettings`).  
- `VibeScribe/Utils/` – Logging, audio helpers, security helpers, chunking, URL builder.  
- `VibeScribe/Sources/` – Cross-cutting extensions (e.g., `TimeInterval.clockString`).  
- Recorded media and caches are written to Application Support via `AudioUtils`.

## Core Workflows
- **Recording pipeline** – `CombinedAudioRecorderManager` orchestrates mic + system capture, merges sources via `AudioUtils`, and drives the Recording Overlay UI.  
- **File import** – `AudioFileImportManager` handles drag-and-drop, format conversion, duration validation, SwiftData persistence, and UI notifications.  
- **Processing pipeline** – `RecordProcessingManager` enqueues transcription (prefers SSE streaming via `WhisperTranscriptionManager`, falls back to polling) and summarization (chunking through `TextChunker`, OpenAI-compatible chat calls, optional auto-title).  
- **Playback & review** – `AudioPlayerManager`, waveform caching, and `RecordDetailView` provide scrubber, speed cycling, inline rename/download, and manual retry controls.  
- **Settings & discovery** – `SettingsView` edits `AppSettings`, tests endpoints, and fetches model lists through `ModelService`.

## Build & Run
- Open `VibeScribe.xcodeproj` and run the **VibeScribe** scheme on macOS.  
- CLI build: `xcodebuild -scheme VibeScribe -configuration Debug -destination 'platform=macOS' build`.  
- UI tests live in `VibeScribeUITests/`; add unit XCTest targets under `VibeScribeTests/` when you introduce coverage.

## Coding Guidelines
- Swift 5+, four-space indentation, filenames match primary types.  
- Prefer idiomatic SwiftUI and async/Combine patterns already used in managers.  
- Use `Logger` for diagnostics instead of `print`; categories cover audio, UI, transcription, LLM, etc.  
- Follow existing data flow: interact with `Record` and `AppSettings` through SwiftData contexts, post `Notification` events when updating UI state.  
- Keep whitespace clean and stick to ASCII unless files already contain Unicode icons.

## Delivery Workflow (Mandatory)
- Any behavior change must start with **user-flow test-case design**, not code changes.
- Before implementation, describe the flow in `VibeScribeUITests/TEST_CASES.md` with:
  - Preconditions,
  - User steps,
  - Expected result,
  - Target UI test method name(s).
- Only after test-case update is done, implement or update UI tests, then implement production code.
- For UI behavior changes, **UI tests are mandatory**. A UI behavior task without UI-test delta is considered incomplete.
- New UI functionality must be designed for testability from day one:
  - stable `accessibilityIdentifier` for interactive controls,
  - deterministic states for UI testing (`VIBESCRIBE_UI_TESTING`, mock pipeline hooks),
  - no hidden timing-only dependencies when a deterministic signal can be exposed.

## Testing & Debugging Notes
- Add deterministic unit tests for pure logic (`AudioUtils`, `TextChunker`, URL building) first.  
- UI flows rely on SwiftData and AppKit status bar integration—use previews or lightweight harnesses where possible.  
- A debug toggle (`simulateEmptyRecordings`) lives in `AppStorage`; gate debug-only code with `#if DEBUG`.
- Use `./scripts/run_test_sets.sh` to run common UI test subsets:
  - `ui-smoke` (fast baseline),
  - `ui-core` (main non-mock flows),
  - `ui-mock` (mock pipeline flows),
  - `ui-all` (full UI suite),
  - `ui-class <ClassName>` / `ui-test <ClassName/testMethod>` for targeted runs.
- **UI test source of truth**: `VibeScribeUITests/TEST_CASES.md` is the canonical contract for UI test inventory, per-class counts, and accessibility coverage scope for fast UI automation.
- **Test case sync rule**: `VibeScribeUITests/TEST_CASES.md` must stay in sync with all UI test sources under `VibeScribeUITests/*.swift`. Sync means: (1) every `func test*` method has a row in markdown, (2) every row maps to an existing method, (3) header total matches `grep -Rho 'func test[A-Za-z0-9_]*' VibeScribeUITests/*.swift | wc -l`, (4) per-class counts match real class method counts.
- **Accessibility coverage rule**: every ID attached through `.accessibilityIdentifier(AccessibilityID.*)` must be either mirrored in UI test IDs (from `VibeScribeUITests/*.swift`) or explicitly listed in `TEST_CASES.md` out-of-scope section with a reason.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pfrankov/vibe-scribe](https://github.com/pfrankov/vibe-scribe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
