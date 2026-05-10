---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

OpenLess is a native macOS menu-bar voice-input layer (SwiftPM, macOS 15+, Swift 5.9). User holds/toggles a global hotkey, speaks, and the dictated text is polished and inserted at the current cursor in any app. The product principles, state machine, and module list live in `docs/openless-development.md` and `docs/openless-overall-logic.md` — read those before changing product behavior.

## Build, Run, Test

The app must run as a bundled `.app` to receive TCC entitlements (Accessibility, Microphone, AppleEvents). Running `swift run` directly will leave permission prompts broken.

```bash
# Library/test build (no bundle, fastest)
swift build
swift test
swift test --filter OpenLessCoreTests.PolishModeTests/<method>   # single test

# Full app: builds release, assembles build/OpenLess.app, ad-hoc codesigns,
# kills the running instance, and resets TCC by default
./scripts/build-app.sh

# Keep existing TCC approvals across rebuilds
RESET_TCC=0 ./scripts/build-app.sh

# Launch
open build/OpenLess.app
# Useful launch args (handled in AppDelegate.runLaunchActions)
open build/OpenLess.app --args --open-settings
open build/OpenLess.app --args --start-recording

# Live logs
tail -f ~/Library/Logs/OpenLess/OpenLess.log
```

After a fresh build with `RESET_TCC=1` (default), macOS will re-prompt for Accessibility + Microphone. The app must be **fully quit and relaunched** after granting Accessibility before the global hotkey tap works.

## Architecture

SwiftPM workspace with one executable (`OpenLessApp`) and seven libraries. Every library depends only on `OpenLessCore` — there are no library-to-library deps. `OpenLessApp` is the only place that wires them together (`DictationCoordinator`).

```
OpenLessCore        // Pure value types: DictationSession, PolishMode, HotkeyBinding,
                    //   AudioConsumer protocol, RawTranscript/FinalText, errors.
                    //   No AppKit. Every other target imports this.
OpenLessHotkey      // CGEventTap-based modifier-key monitor. Emits .toggled / .cancelled
                    //   on an AsyncStream. Requires Accessibility permission.
OpenLessRecorder    // AVAudioEngine → 16 kHz mono Int16 PCM, RMS level callback,
                    //   pushes Data chunks into an AudioConsumer.
OpenLessASR         // Volcengine streaming ASR over WebSocket (binary framing in
                    //   VolcengineFrame). Implements AudioConsumer so the recorder
                    //   feeds it directly.
OpenLessPolish      // Doubao (Ark) chat-completions client. Mode-driven prompts in
                    //   PolishPrompts.
OpenLessInsertion   // TextInserter: AX focused-element write first, then clipboard
                    //   + simulated Cmd+V, then copy-only fallback.
OpenLessPersistence // CredentialsVault (Keychain), HistoryStore (JSON in Application
                    //   Support), UserPreferences (UserDefaults).
OpenLessUI          // Capsule SwiftUI view + state enum (no window plumbing).
OpenLessApp         // AppDelegate, MenuBarController, SettingsWindowController,
                    //   CapsuleWindowController, Log, BufferingAudioConsumer,
                    //   DictationCoordinator (the state machine).
```

### Dictation pipeline

`DictationCoordinator` (`Sources/OpenLessApp/DictationCoordinator.swift`) is the single owner of session state. It is `@MainActor` and uses a one-bit `inSession` toggle driven by `HotkeyMonitor.events`:

```
.toggled (1st)  →  beginSession:
                     Recorder.start (mic, level callback → capsule)
                     VolcengineStreamingASR.openSession (WebSocket)
                     BufferingAudioConsumer.attach(asr)   // any pre-connect audio is replayed

.toggled (2nd)  →  endSession:
                     Recorder.stop, ASR.sendLastFrame, awaitFinalResult
                     DoubaoPolishClient.polish(raw, mode)
                     TextInserter.insert(final)           // .inserted | .copiedFallback
                     HistoryStore.save(DictationSession)
                     CapsuleWindowController hides after delay

.cancelled      →  ASR.cancel, Recorder.stop, capsule .cancelled
```

Important behaviors:
- **Polish/ASR fallbacks are silent**: if Ark creds are missing or polish fails, the raw transcript is inserted; if Volcengine creds are missing, a mock pipeline runs and copies a placeholder. Don't add hard errors here — the contract is "user's words don't get lost."
- **`BufferingAudioConsumer`** queues PCM until the WebSocket is ready, then drains. Recorder always pushes to it; ASR is attached after `openSession` resolves.
- **Hotkey is toggle-only**, not press-and-hold. The monitor yields one `.toggled` per modifier-key keydown; the coordinator interprets odd/even.

### Permissions, credentials, on-disk state

- **TCC**: `Info.plist` declares `NSMicrophoneUsageDescription` and `NSAppleEventsUsageDescription`. Accessibility is requested at first run via `AXIsProcessTrustedWithOptions`. `LSUIElement = true` (menu-bar only, no Dock icon).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [appergb/openless](https://github.com/appergb/openless) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-27 -->
