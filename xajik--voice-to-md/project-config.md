---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test Commands

The Xcode project is **generated from `project.yml`** — it is gitignored and must be regenerated after any structural change.

```bash
make setup          # Install xcodegen + generate .xcodeproj (first time)
make generate       # Re-generate after editing project.yml
make check          # Verify whisper-cli, ffmpeg, xcodegen are installed

make build          # Release build + sign (auto-detected identity, see below)
make build-debug    # Debug build + sign
make run            # Release build + open the app
make test           # Run all unit tests
make lint           # SwiftLint (optional; skipped if not installed)
make clean          # Remove .build/ and .xcodeproj
make install        # deps + build + copy to /Applications (install.sh delegates here)
```

**Run a single test class:**
```bash
xcodebuild -scheme SpeechToMarkdown -configuration Debug -derivedDataPath .build \
  -destination 'platform=macOS' \
  -only-testing:SpeechToMarkdownTests/TranscriptBufferTests \
  CODE_SIGN_IDENTITY="" CODE_SIGNING_REQUIRED=NO CODE_SIGNING_ALLOWED=NO test
```

**Signing:** ad-hoc signatures change every build and silently invalidate macOS TCC grants (microphone, Accessibility). `make build` therefore auto-detects the first "Apple Development" identity in the keychain and signs with it (hardened runtime + entitlements); ad-hoc (`-s -`) is only the fallback when no identity exists. Override with `make build SIGN_IDENTITY="…"` or force ad-hoc with `SIGN_IDENTITY=-`.

## What the app does

Menu-bar macOS app, two flows, both local-only (no cloud services):

1. **Global dictation** — hotkey **⌘⌥]** anywhere: record → whisper.cpp transcription → text typed at the cursor via CGEvent. A Spotlight-style floating panel (`DictationHUDView` in a non-activating `NSPanel`) shows listening/transcribing state.
2. **Agent mode** — window with a markdown editor: record → whisper → 30-word buffer → streaming chat-completions call to a **local OpenAI-compatible LLM server** (omlx/llama.cpp/LM Studio, default `http://127.0.0.1:8000/v1`) that formats the transcript into a live-updating document. Output format is selectable in the HUD control panel: **md** (default), **txt**, or **html** (`OutputFormat` enum; each case carries prompt expectations + a worked example).

## Architecture

### Entry point and UI ownership

`SpeechToMarkdownApp.swift` is `@main` but its `body` only exposes an empty `Settings` scene. All real setup is in `AppDelegate`: status bar item, `SessionCoordinator`, `GlobalDictationManager`, the dictation `NSPanel`, and windows created on demand. **Any window created here must set `isReleasedWhenClosed = false` and be retained in a property** — otherwise AppKit + ARC double-release it on close and crash in the close animation.

### Agent mode pipeline

`SessionCoordinator` (`@MainActor ObservableObject`) owns everything:
- `AudioCaptureService` (AVAudioEngine 16 kHz mono) → PCM buffers are **accumulated into ~4 s chunks** (`transcribeChunkSeconds`) and transcribed **serially** via a FIFO task chain (`enqueueAfterTranscription`). Never transcribe per tap buffer — whisper-cli reloads the model per invocation.
- `WhisperService` (whisper-cli subprocess). `isNoiseOnly()` filters annotation-only chunks like `(wind blowing)` before they reach the buffer.
- `TranscriptBuffer` actor — two-buffer accumulation (30-word flush threshold; pending queue while the LLM is busy).
- `LocalLLMService` — `listModels()` + streaming `formatTranscript()`/`editDocument()`/`appendTranscript()` (SSE); `cleanOutput()` strips `<think>` blocks and code fences. Partial output streams straight into `coordinator.markdown`. System prompts are per-mode **functions** taking an `OutputFormat` — each appends the format's `promptExpectations` (rules + example) and ends with `/no_think`. Payload key is `current_document` (format-neutral).
- `OutputFormat` (Models/) — txt/md/html; drives the system prompt, the HUD dropdown, and the session file extension. Persisted in `BackendSettings` (`stmd.outputFormat`); snapshotted per flush alongside `mode` so mid-stream changes can't corrupt an in-flight request. Changing it mid-session renames the document file and restarts the `FileWatcher` (`migrateSessionDocument()`).
- `BackendSettings` — UserDefaults-backed base URL + model name (empty = auto-pick first model from `/v1/models`) + output format.
- Session files: `~/.stmd/speech-to-markdown/{unix_ms}/{id}.txt` (append-only raw) and `{id}.{txt|md|html}` (rewritten; extension follows the output format — `STMDSession.docPath`). When the format is `txt`, `docPath` and the raw transcript path are the same file.
- `STMDFileManager.listSessions()` scans `speech-to-markdown/` for the recent-sessions list (`SessionListing`, Models/): format is detected by probing for a second `{id}.md`/`{id}.html` file alongside the always-present `{id}.txt` (no second file ⇒ `txt`). Directory-parameterized as `listSessions(in:)` for testing against a temp dir instead of the real `~/.stmd` tree.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xajik/voice-to-md](https://github.com/xajik/voice-to-md) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
