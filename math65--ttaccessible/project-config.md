---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Build Commands

```bash
# Debug build (CLI)
xcodebuild -project App/ttaccessible.xcodeproj -scheme ttaccessible -configuration Debug build

# Run the unit-test suite (ttaccessibleTests target)
xcodebuild test -project App/ttaccessible.xcodeproj -scheme ttaccessible -destination 'platform=macOS'

# Launch built app (Debug)
open ~/Library/Developer/Xcode/DerivedData/ttaccessible-*/Build/Products/Debug/ttaccessible.app

# Release build + zip artifact (staged for Sparkle appcast)
./build.sh

# Release + Developer ID sign + notarize + staple
./build.sh --notarize

# Same as --notarize, plus push a draft GitHub release
./build.sh --release

# Beta-channel release: appcast item tagged <sparkle:channel>beta</sparkle:channel>
# (only delivered to users who enabled "Include beta versions") + GitHub prerelease
./build.sh --release --beta

# Regenerate the Apple Help Book after editing Help/Source/**.md
./scripts/build-help-book.sh <marketing-version> <build-number>
./scripts/build-help-book.sh --dev     # timestamp version, defeats the helpd cache
```

`build.sh` re-signs the Xcode-built .app with the `Developer ID Application` cert (the project itself still builds with Apple Development for convenience). Requires the notarytool keychain profile `ttaccessible-notary` to be stored (see notarization setup memory).

An XCTest unit-test target (`ttaccessibleTests`, file-system synchronized group) covers
pure/deterministic logic only: the gain dB↔% and user-volume↔% curves, `clampGainDB`, and
Codable migrations of preference structs. Run via the `xcodebuild test` command above.

The tests deliberately do NOT touch AppKit UI, CoreAudio, or the TeamTalk SDK runtime —
verify those by building and running the app manually.

## Language

The user speaks French. Respond in French when communicating. Code, comments, and commit messages stay in English.

## Architecture

**macOS AppKit app** with SwiftUI preference panes. Built for accessibility (VoiceOver). Localized in English and French.

### TeamTalk SDK

The app wraps the TeamTalk 5 C library (`Vendor/TeamTalk/libTeamTalk5.dylib`) via a bridging header. The SDK instance is a raw `UnsafeMutableRawPointer` managed by `TeamTalkConnectionController`. All SDK calls (`TT_*` functions) must happen on the serial dispatch queue `com.math65.ttaccessible.teamtalk`.

**PortAudio probe patch (mandatory, applied at build time)**: The vendored dylib is binary-patched by `scripts/patch-sdk-portaudio.py` so PortAudio's `IsFormatSupported()` returns immediately — this removes the ~13 s startup device probe (`TT_InitTeamTalkPoll` opening/closing a CoreAudio stream per device × per standard rate). Safe because the app only ever uses the TeamTalk virtual device, never a real PortAudio device. The patcher is symbol-driven, idempotent, and fail-loud (aborts if the prologue isn't a known-good unpatched or already-patched byte sequence). It runs in TWO places: (1) `scripts/download-sdk.sh` after a fresh download, and (2) a **`Patch TeamTalk SDK (PortAudio probe)`** run-script build phase (first in the app target, before Sources/Embed — PR #27) so no build can ship an unpatched dylib regardless of how the gitignored dylib landed on the machine. **beta.9 and earlier shipped an unpatched dylib** because the patch was only coupled to a fresh download; the build phase is the permanent fix (first shipped in beta.10). If a future SDK bump changes the `_IsFormatSupported` prologue, the patcher aborts the build — update `ORIGINAL_PROLOGUES` in the script after verifying the new bytes by hand.

**Documented exceptions to the serial-queue rule** (both rely on the SDK's per-instance internal locking, validated in the field — not a violation):
- **Prewarm** (`+Connection`): a new instance is created on a background queue (`TT_InitTeamTalkPoll`) to hide the sound-system probe latency.
- **`AudioBlockPump`** (PR #26, beta.9): per-user voice/media audio blocks are drained via `TT_AcquireUserAudioBlock` / `TT_ReleaseUserAudioBlock` on the pump's own 10 ms user-interactive timer queue, NOT the message loop. Rationale: acquiring blocks inside the 20 ms message-loop drain shared the serial queue with channel-tree publish/history/state work, so one slow tick in a crowded channel starved every mix source at once (choppy for everyone, zero ring underflows). The muxed AEC-reference stream (pre-14.2 fallback) stays on the message loop. `stop()` is synchronous (`queue.sync`) so no acquire is in flight when the instance is torn down.

**Critical**: The app does NOT use `TT_InitSoundInputDevice` / `TT_EnableVoiceTransmission` for microphone capture because the SDK's direct audio path causes audio saturation/crackling. Instead, it uses a dual-path capture engine (`AdvancedMicrophoneAudioEngine`) that captures audio, applies input gain, converts to Int16 PCM, and injects chunks via `TT_InsertAudioBlock` into the TeamTalk virtual sound device (`TT_SOUNDDEVICE_ID_TEAMTALK_VIRTUAL`).

### Core Components


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [math65/ttaccessible](https://github.com/math65/ttaccessible) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
