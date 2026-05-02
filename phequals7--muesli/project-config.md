---
trigger: always_on
description: > **Post-compaction recovery:** PreCompact hooks auto-generate context handover files at `Context/handoff-summary-YYYY-MM-DD-<slug>.md`. After compaction, read the latest handoff file in `Context/` to restore session memory and resume work.
---

> **Post-compaction recovery:** PreCompact hooks auto-generate context handover files at `Context/handoff-summary-YYYY-MM-DD-<slug>.md`. After compaction, read the latest handoff file in `Context/` to restore session memory and resume work.

# Muesli

Local-first macOS app for **dictation** and **meeting transcription** on Apple Silicon. All speech-to-text runs on-device via CoreML/Neural Engine. Native Swift/AppKit — no Electron, no Python runtime, no cloud STT costs.

**Status:** Live and public. Available at [GitHub Releases](https://github.com/pHequals7/muesli/releases). Signed, notarized, stapled.

## What It Does

- **Dictation:** Hold hotkey → speak → release → text pasted at cursor (~0.13s with Parakeet)
- **Meeting transcription:** Captures mic (You) + system audio (Others) → VAD-driven chunking → speaker diarization → AI-powered meeting notes
- **Meeting export:** Export notes or transcript as PDF (paginated US Letter) or Markdown via `MeetingExporter.swift`
- **Screen context:** Accessibility API captures app name + text around cursor for dictation context-awareness (opt-in, off by default)
- **7 ASR models:** Parakeet v3/v2, Whisper Small/Medium/Large Turbo, Qwen3 ASR, Nemotron Streaming
- **3 summarization backends:** OpenAI API key, OpenRouter API key, ChatGPT OAuth (subscription-based)
- **Camera-based meeting detection:** Requires mic + camera + recognized meeting app (camera alone won't trigger)
- **Join & Record:** Extract meeting URLs from calendar events (Zoom, Meet, Teams, Webex, Chime, FaceTime), split button with "Join & Record" / "Join Only" / "Record Only", platform icons in notifications
- **Google Calendar integration:** Coming Up section, status bar, pre-meeting countdowns, event-driven notifications via `EKEventStoreChangedNotification`
- **Meeting templates:** Built-in and custom templates for structured meeting notes

## Building

### Production build (signed, installed to /Applications)
```bash
./scripts/build_native_app.sh
```

### Dev/test build (isolated, unsigned)
```bash
./scripts/dev-test.sh                  # Build MuesliDev.app (separate bundle ID, separate data)
./scripts/dev-test.sh --clean          # Wipe dev data, fresh onboarding
./scripts/dev-test.sh --reset          # Re-run onboarding, keep dev data
./scripts/dev-seed-from-prod.sh        # Copy production DB/config into MuesliDev safely
./scripts/dev-reset-permissions.sh     # Reset macOS privacy permissions for MuesliDev
```

MuesliDev uses bundle ID `com.muesli.dev` and stores data at `~/Library/Application Support/MuesliDev/`. Production data is never touched.

### Tests
```bash
swift test --package-path native/MuesliNative    # 396 tests across 65 suites
```

### Onboarding testing
```bash
# Reset onboarding flag without losing data:
python3 -c "import json; p='$HOME/Library/Application Support/MuesliDev/config.json'; c=json.load(open(p)); c['has_completed_onboarding']=False; json.dump(c,open(p,'w'),indent=2)"
# Reset macOS permissions:
./scripts/dev-reset-permissions.sh
# Then:
./scripts/dev-test.sh
```
Note: config JSON uses snake_case keys (`has_completed_onboarding`, not `hasCompletedOnboarding`).

## CI/CD Pipeline

### Pull Requests
- **CI workflow** (`.github/workflows/ci.yml`): macOS 15 runners
  - `changes` → `build` → `cli-smoke` → `ci-gate` (required check)
- **Claude Code Review** — reviews every PR automatically
- **Greptile** — reviews every PR automatically
- **Vercel** — scoped to `site/` only
- **Concurrency** — stale CI runs auto-cancelled on new pushes

### Releases
```bash
./scripts/release.sh                   # Auto-increments version
./scripts/release.sh 1.0.0             # Explicit version
```
**Critical:** Staple the app bundle BEFORE creating the DMG, otherwise Gatekeeper rejects.

### Signing & Notarization
- Developer ID: `Pranav Hari Guruvayurappan (58W55QJ567)`
- Bundle ID: `com.muesli.app`
- Notary profile: `MuesliNotary` (Keychain)

## Key Architecture

```
native/MuesliNative/Sources/
├── MuesliNativeApp/              # Main app (~50 Swift files)
│   ├── MuesliController.swift    # Central orchestrator — dictation, meetings, onboarding, state
│   ├── TranscriptionRuntime.swift # Routes to ASR backends, post-processing, VAD + diarization
│   ├── FluidAudioBackend.swift   # Parakeet TDT on ANE
│   ├── Qwen3AsrBackend.swift     # Qwen3 ASR on ANE (macOS 15+)
│   ├── Qwen3PostProcessor.swift  # On-device GGUF LLM for dictation cleanup (opt-in)
│   ├── WhisperKitBackend.swift   # Whisper on CoreML/ANE via WhisperKit
│   ├── ScreenContextCapture.swift # AX-based app context for dictation + meetings
│   ├── MeetingExporter.swift     # PDF/Markdown export with NSPrintOperation
│   ├── OnboardingView.swift      # 7-step onboarding with real permission polling + dictation test
│   ├── OnboardingProgress.swift  # Crash-safe onboarding state persistence
│   ├── MeetingSession.swift      # Meeting lifecycle + diarization + screen context
│   ├── MeetingSummaryClient.swift # OpenAI / OpenRouter / ChatGPT summarization
│   ├── SystemAudioRecorder.swift # ScreenCaptureKit SCStream for system audio

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pHequals7/muesli](https://github.com/pHequals7/muesli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
