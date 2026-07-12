---
trigger: always_on
description: Briefing for AI coding agents working on this repo. Complements
---

# AGENTS.md

Briefing for AI coding agents working on this repo. Complements
`README.md` (for humans) and `CONTRIBUTING.md` (for contributors).

## Project shape

Presspeech is a **single-file Swift menu-bar app** for push-to-talk
dictation on Apple Silicon Macs. The whole app is
`swift/Sources/Presspeech/main.swift` (currently about 12k lines). The `PresspeechApp`
@MainActor class owns the menu bar, settings UI, recording loop,
and update flow; surrounding it in the same file are the
single-responsibility types it composes (`Settings`, `Permissions`,
`HotkeyListener`, `AudioCapture`, `TranscriptionWorker` actor,
`TranscriptCorrector`, `FillerWordRemover`, `TextInserter`,
`UpdateCheck`, `TCC`, etc.). The hot path is:

1. A Quartz `CGEventTap` (`HotkeyListener`) catches the user's hotkey.
   Modifier keys are diffed in `flagsChanged`; regular keys come in
   via `keyDown` / `keyUp`. The chosen key is suppressed so it can't
   fire other shortcuts.
2. While held, `AVAudioEngine` taps the input device and
   `AVAudioConverter` resamples to 16 kHz mono Float32 (`AudioCapture`,
   `NSLock`-protected — see *Swift concurrency model* below).
3. On release the Float32 buffer is handed to a
   `TranscriptionWorker` actor that owns FluidAudio's `AsrManager`.
   Parakeet TDT v3 runs on the **Apple Neural Engine** via CoreML.
4. The transcript hits `NSPasteboard`, `Cmd+V` is posted via
   `CGEvent.post`, `NSAppleScript` unmutes system audio, and the
   "Pop" `NSSound` plays.

### Key files

| Path | Purpose |
|---|---|
| `swift/Sources/Presspeech/main.swift` | The entire app. `// MARK: -` section comments tag the major regions (Constants, Text correction transfer, Correction sync path safety, Model registry hardening, Speech model integrity, Audio input devices, Logger, Settings, Permissions, Hotkey listener, Audio capture, Transcription worker, Transcript corrections, Filler word removal, Text insertion, System audio mute, Sounds, Bundle version helpers, Diagnostics, TCC recovery, Update check, App). |
| `swift/Package.swift` | SwiftPM manifest. `.macOS("14.0")` platform target, single FluidAudio dependency. **No `resources:` declaration** — resources live outside the target on purpose (see *Resource bundling* below). |
| `swift/Info.plist` | Canonical Info.plist for both dev and release builds. `CFBundleIdentifier com.local.presspeech`, `LSMinimumSystemVersion 14.0`. `dev-run.sh` signs with the same Developer ID cert and identifier as the Cask, so TCC grants from the production install carry over to the dev binary automatically. |
| `swift/Resources/presspeech-menubar.png` (+ `@2x`) | Template menu-bar icon. Copied into `Contents/Resources/` by `dev-run.sh` and `ship-swift.sh`. |
| `swift/dev-run.sh` | Local iteration loop: `swift build` → wrap binary in `/tmp/Presspeech-dev.app` → sign with Developer ID + hardened runtime + production entitlements → relaunch. |
| `entitlements.plist` | Hardened-runtime entitlements. Just two keys: `device.audio-input` (what Tahoe 26 checks before exposing the app in Privacy & Security → Microphone) and `device.microphone` (legacy sandbox fallback for macOS 14–25). Anything new expands TCC surface — justify before adding. |
| `ship-swift.sh` | One-command release: version bump in Info.plist → build → sign → notarise → ditto-zip → tag → push → `gh release create` → bump and verify sibling Homebrew Cask. |
| `scripts/update-model-manifest.py` | Regenerates the pinned SHA-256 manifest for the Parakeet v3 CoreML model files when intentionally updating the upstream model commit. |
| `icon/` | SVG sources (`hero.svg`, `latency.svg`, `presspeech.svg`, etc.), `Presspeech.icns`, menu-bar PNGs, `make-icons.sh`. |
| `experiments/swift-bench/` | Standalone ASR latency benchmark used to validate FluidAudio against alternatives (Apple SpeechAnalyzer, presspeech-mlx) on the same audio. Re-run when bumping FluidAudio or evaluating a backend swap. |

## Build & test

```sh
# Debug build + run as a signed .app (the canonical dev loop)
cd swift
./dev-run.sh

# Run the in-binary self-test suite (no UI, exits 0/1)
swift run Presspeech --self-test all

# Release dry-run: build + sign + entitlement check + zip; skips notarise/staple and git/tag/release/cask
# (ship-swift.sh lives at the repo root; this block is still in swift/)
../ship-swift.sh --dry-run

# Tail logs (same file for dev + Cask install — bundle ids match)
tail -f ~/Library/Logs/Presspeech.log
```

There is no separate unit-test target — pure logic instead exposes
itself through the `--self-test` lane on the same binary. Suites:
`hotkey` (transition state machine), `readiness` (permission-rollup
state machine), `paste` (suffix formatting), `history`
(`RecentTranscriptLimit` slicing), `corrections` (transcript
correction apply/merge), `fillers` (filler-word removal),
`audio-level` (level metering), `audio-conversion` (offline
conversion/downmix rules), `audio-input` (input-device filtering),
`model-status` (speech-model startup labels), `audio-route`
(route-change decisions), `recording-lifecycle` (release/post-
processing decisions), `power-state` (sleep/wake recovery decisions),
`model-integrity` (speech-model hash
verification), `update` (GitHub update parsing and update-helper

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rcourtman/presspeech](https://github.com/rcourtman/presspeech) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
