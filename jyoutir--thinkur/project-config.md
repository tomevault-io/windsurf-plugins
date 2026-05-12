---
trigger: always_on
description: > **NEVER run `xcodebuild`, `swift build`, or ANY build/compile command from `~/Downloads/thinkur`.** macOS Sequoia adds `com.apple.macl` + `com.apple.provenance` extended attributes when build tools touch files in `~/Downloads`. These xattrs are enforced at kernel level and **cannot be removed** — even `sudo xattr -cr` fails. The entire directory becomes permanently inaccessible to Terminal, editors, git, and all CLI tools. The only recovery is to delete the folder in Finder and re-clone. See `
---

# thinkur

> **NEVER run `xcodebuild`, `swift build`, or ANY build/compile command from `~/Downloads/thinkur`.** macOS Sequoia adds `com.apple.macl` + `com.apple.provenance` extended attributes when build tools touch files in `~/Downloads`. These xattrs are enforced at kernel level and **cannot be removed** — even `sudo xattr -cr` fails. The entire directory becomes permanently inaccessible to Terminal, editors, git, and all CLI tools. The only recovery is to delete the folder in Finder and re-clone. See `docs/building.md` for how to build safely.

Offline macOS menu bar voice typing app. Tap a hotkey to start recording, tap again to stop, transcribed text pastes at cursor. 100% local — WhisperKit on CoreML, no cloud.

## Build

Use Xcode (Cmd+R) for Debug builds, or the Xcode MCP bridge. Release: `scripts/build-dmg.sh`.

```sh
# If new source files were added, regenerate Xcode project first:
xcodegen generate
```

## Test

```sh
swift test
```

> `xcodebuild test` has a bootstrapping issue — always use `swift test`.

## Run

Run from Xcode (Cmd+R) or launch `/Applications/thinkur.app` directly. The app lives in the menu bar (no dock icon).

## Project Structure

```
thinkur/
├── project.yml                          ← xcodegen spec (source of truth for .xcodeproj)
├── Package.swift                        ← SPM deps
├── Sources/thinkur/
│   ├── thinkurApp.swift                 ← @main, MenuBarExtra
│   ├── Resources/
│   │   ├── Info.plist                   ← LSUIElement, usage descriptions
│   │   └── thinkur.entitlements         ← audio input, bluetooth, network
│   ├── Core/
│   │   ├── AppRuntimeConfiguration.swift        ← reads plist keys for dev/release split
│   │   ├── AppState/SharedAppState.swift        ← @Observable, single source of truth
│   │   ├── DI/ServiceContainer.swift            ← dependency injection container
│   │   ├── DI/ViewModelFactory.swift            ← creates view models with injected deps
│   │   ├── Coordinators/                        ← ModelLoadCoordinator, HotkeyCoordinator, RecordingCoordinator
│   │   ├── AudioCaptureManager.swift            ← AVAudioEngine → 16kHz mono Float32 + RMS
│   │   ├── TranscriptionEngine.swift            ← WhisperKit wrapper (large-v3)
│   │   ├── HotkeyManager.swift                  ← CGEvent tap, customizable hotkey
│   │   ├── TextInsertionService.swift           ← clipboard save → Cmd+V paste → restore
│   │   ├── PermissionManager.swift              ← Accessibility, Mic, Input Monitoring
│   │   ├── Processors/                          ← 9 post-processing processors
│   │   ├── PostProcessing/Rules/                ← static data (word lists, patterns)
│   │   ├── PostProcessing/Matchers/             ← reusable matching logic
│   │   ├── PostProcessing/Models/               ← ReplacementRule, PauseThresholds, etc.
│   │   ├── PostProcessing/Utilities/            ← RegexCache, TextMutator, NLTaggerHelper
│   │   └── Data/SwiftDataContainerFactory.swift ← SwiftData store setup
│   ├── UI/                                      ← SwiftUI views, floating panels
│   └── Utilities/                               ← Constants, Logger, FrontmostAppDetector
├── Tests/thinkurTests/
│   ├── Processors/    ← post-processing tests
│   ├── Services/      ← service layer tests
│   ├── ViewModels/    ← view model tests
│   ├── Mocks/         ← test doubles
│   ├── Integration/   ← integration tests
│   └── Utilities/     ← utility tests
└── scripts/
    ├── release.sh              ← orchestrator: prepare (build+stage) or publish
    ├── release-preflight.sh    ← pre-flight checks
    ├── bump-version.sh         ← version bump
    ├── build-dmg.sh            ← archive → sign → notarize → DMG
    ├── stage-release.sh        ← create/update draft GitHub Release with DMG
    ├── publish-appcast.sh      ← generate appcast → push → publish draft
    ├── bootstrap-release-tools.sh ← cache Sparkle tools to ~/.cache/thinkur/
    ├── install-dev-app.sh      ← post-build: copy Dev app to ~/Applications
    ├── dev-reset-permissions.sh ← manual TCC reset for dev bundle ID
    ├── reset-for-testing.sh    ← wipe local state for testing
    └── lib/
        └── release-common.sh   ← shared helpers for release scripts
```

## Architecture

```
Hotkey (CGEvent tap) → AudioCaptureManager (AVAudioEngine 16kHz)
                      → TranscriptionEngine (WhisperKit large-v3)
                      → TextPostProcessor (9-stage pipeline)
                      → TextInsertionService (clipboard Cmd+V)
                      → FloatingIndicatorPanel (waveform overlay while recording)
```

- **SharedAppState** is the single source of truth for app state, model readiness, transcription
- **ServiceContainer + ViewModelFactory** provide dependency injection
- **Tap-to-toggle**: hotkey once to start, again to stop and transcribe

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jyoutir/thinkur](https://github.com/jyoutir/thinkur) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
