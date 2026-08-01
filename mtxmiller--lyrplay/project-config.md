---
trigger: always_on
description: **LyrPlay** is an iOS SwiftUI app that implements a SlimProto client for streaming audio from Logitech Media Server (LMS). It's essentially a **Swift version of squeezelite** — a Squeezebox player replacement with native FLAC support, CarPlay, Siri, and gapless playback.
---

# CLAUDE.md

## Project Identity

**LyrPlay** is an iOS SwiftUI app that implements a SlimProto client for streaming audio from Logitech Media Server (LMS). It's essentially a **Swift version of squeezelite** — a Squeezebox player replacement with native FLAC support, CarPlay, Siri, and gapless playback.

- **Version state**: build/version live in `LMS_StreamTest.xcodeproj/project.pbxproj` (`MARKETING_VERSION`, `CURRENT_PROJECT_VERSION`). Per-release status (in dev / submitted / live) tracked in the Obsidian wiki under `Releases/`.
- **Bundle ID**: `elm.LMS-StreamTest` (preserved for App Store continuity — never change this)
- **Display Name**: LyrPlay
- **Local Folder**: `LMS_StreamTest` (intentional — don't rename)
- **GitHub**: https://github.com/mtxmiller/LyrPlay
- **Deployment Target**: iOS 15.6+ (affects which APIs are available)

## Build Commands

```bash
pod install                          # Install dependencies (required after clone)
xcodebuild -workspace LMS_StreamTest.xcworkspace -scheme LMS_StreamTest -configuration Debug build
xcodebuild -workspace LMS_StreamTest.xcworkspace -scheme LMS_StreamTest clean
```

**Always use `LMS_StreamTest.xcworkspace`**, never `.xcodeproj` (CocoaPods requirement).

For the CLI build → install → launch loop on a connected iPhone, see the wiki at `Setup/iPhone Build Workflow.md`. Personal device IDs are kept in user-local Claude memory, not committed.

**Test LMS server**: `192.168.1.8` (default ports — 9000 JSON-RPC, 3483 SlimProto) is available on-network for debugging and exercising new features.

## Testing & Verification

Unit test targets exist on both platforms (Swift Testing + XCTest). Run them before claiming a change works:

```bash
# tvOS — use -only-testing: the UITests runner has a known dlopen failure (bd LMS_StreamTest-39m)
xcodebuild test -workspace LMS_StreamTest.xcworkspace -scheme LMS_StreamTest-tvOS \
  -destination 'platform=tvOS Simulator,name=Apple TV 4K (3rd generation)' \
  -only-testing:LMS_StreamTest-tvOSTests

# iOS — device name must exist on the LATEST installed iOS runtime
# (xcodebuild implies OS:latest; a name that only exists on an older runtime
# fails with "Unable to find a device matching"). Check: xcrun simctl list devices available
xcodebuild test -workspace LMS_StreamTest.xcworkspace -scheme LMS_StreamTest \
  -destination 'platform=iOS Simulator,name=iPhone 17' \
  -only-testing:LMS_StreamTestTests
```

The full iOS unit suite passes. (The 3 `PlaybackSessionControllerTests` interruption/route-change tests that used to fail were stale assertions of pre-BASS-migration behavior — local pause/play + manual `setActive` — not an iOS 26 issue; rewritten to assert the current server-command behavior. bd `LMS_StreamTest-u91`, closed.) If unit tests fail, investigate — don't pre-attribute to a known-failing list.

Changes to shared files (`LMS_StreamTest/*.swift` compiled into both targets) must be verified on **both** platforms.

**Server-as-oracle verification**: LyrPlay is a client of an observable server — playback health is machine-checkable over JSON-RPC against the test LMS, no listening required. A healthy pipeline shows: player present in `serverstatus`, `mode == play`, `time` advancing at ~1x wall clock, `playlist_cur_index` changing at expected track boundaries (not early/late). Use these signals to verify playback-adjacent changes end-to-end. Full scenario catalog + assertion library spec: `scripts/smoke/README.md` (implementation tracked in bd epic `LMS_StreamTest-6b1`).

**Still human-only**: audible quality (gapless seams, audio bursts, clicks), CarPlay hardware behavior, real phone-call interruptions, lock-screen timing. Don't claim these verified from a simulator.

**Autonomous pilot loop**: `/pilot` (`.claude/commands/pilot.md`) runs one bd-issue → draft-PR iteration scoped to tvOS/UI work (audio pipeline and recovery files are off-limits to it); `/loop /pilot` keeps it running. It never merges, never closes bd issues, and caps at 3 open draft PRs.

## Issue Tracking

This project uses [bd (beads)](https://github.com/steveyegge/beads) for issue tracking. Use `bd` commands, not markdown TODOs. Run `bd ready --json` for available work, `bd create "title" -t bug|feature|task -p 0-4 --json` to file issues, `bd close <id>` to complete.

## Critical Rules

1. **NO ASSUMPTIONS** — Verify everything against the code and reference repos. Use adversarial review when making changes to critical systems (audio pipeline, SlimProto, recovery).
2. **Never manually manage AVAudioSession** — BASS handles all session lifecycle via `BASS_CONFIG_IOS_SESSION`. Manual management causes silent audio failures and conflicts.
3. **Never add an Intents Extension for Siri** — INPlayMediaIntent is handled in the main app via `SiriMediaHandler` in AppDelegate. This is an App Store validation constraint.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mtxmiller/LyrPlay](https://github.com/mtxmiller/LyrPlay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
