---
trigger: always_on
description: Canonical instructions for AI coding agents working in this repository. Codex, Cursor, and Copilot
---

# AGENTS.md

Canonical instructions for AI coding agents working in this repository. Codex, Cursor, and Copilot
read this file directly; Claude Code reaches it via the `@AGENTS.md` import in `CLAUDE.md`.

## Overview

Prebid Mobile iOS SDK — an open-source header bidding SDK that integrates with Prebid Server to increase ad yield. Version 3.3.0, supports iOS 13+, Swift 5.0+. Distributed via CocoaPods, SPM, and Carthage.

## Agent runbooks (`agents/`)

Detailed procedures live in `agents/` as plain markdown, readable by any agent. Read the relevant
file before acting rather than improvising the commands.

Claude Code additionally loads these as Agent Skills through the committed `.claude/skills`
symlink, which points at `agents/`. `agents/` remains the single source of truth — do not duplicate
files under `.claude/`.

| Task | Read |
|------|------|
| Build the 4 XCFrameworks | `agents/build-sdk/SKILL.md` |
| Run tests, a single test class, or fix post-migration build errors | `agents/xcodebuild/SKILL.md` |
| Review a PR on this repo | `agents/review/SKILL.md` |
| Run SwiftLint | `agents/lint/SKILL.md` |
| CocoaPods install | `agents/pod-install/SKILL.md` |
| Generic iOS/Swift background (not repo-specific; playbook wins on conflicts) | `agents/ios-development/SKILL.md` |
| Generic ObjC → Swift / XCTest migration background (playbook wins on conflicts) | `agents/migration-patterns/SKILL.md` |

## Commands

### Build

```bash
# Build all XCFrameworks (PrebidMobile, GAM, AdMob, MAX) into generated/output/
./scripts/buildPrebidMobile.sh

# Build and publish the SPM release
./scripts/buildPrebidSPM.sh
./scripts/publishSPM.sh
```

Requires CocoaPods installed (`pod` on PATH — GHA `macos-15` ships with it pre-installed). Build output goes to `generated/output/` as `XC<name>.xcframework` (e.g. `XCPrebidMobile.xcframework`). Logs go to `generated/log/prebid_mobile_build.log`. Build uses `Lib-`-prefixed scheme names (`Lib-PrebidMobile`, etc.) to avoid colliding with auto-generated SPM schemes.

### Tests

```bash
# Run PR subset (694 tests) — used on PRs in CI
./scripts/testPrebidMobile.sh --latest --quick

# Run full suite (1111 tests) — used on bump-to branches / run-full-tests label
./scripts/testPrebidMobile.sh --latest

# Run adapter tests (GAM, AdMob, MAX)
./scripts/testPrebidMobileAdapters.sh
```

Flags: `--latest` skips the legacy iOS 13 sanity test (always use locally); `--quick` switches the test plan from `PrebidMobileTests` (full) to `PrebidMobilePRTests` (PR subset). The script creates the `iPhone-16-Pro-PrebidMobile` simulator, runs `build-for-testing` then `test-without-building` with `-retry-tests-on-failure`, then deletes the simulator. Any pre-existing simulator with that name is deleted first.

Test plans: `PrebidMobileTests/PrebidMobileTests.xctestplan` (full, 1111 tests), `PrebidMobileTests/PrebidMobilePRTests.xctestplan` (PR subset, 694 tests).

**When adding new tests:** both plans select by *exclusion*, so a new test class runs on every PR without any registration step. `PrebidMobilePRTests.xctestplan` trims the suite via a `skippedTests` list — check that a new class isn't (prefix-)matched there, and add an entry only if you deliberately want it excluded from the PR subset.

To run a single test class:
```bash
# Step 1 — build once
xcodebuild \
  -workspace PrebidMobile.xcworkspace \
  -scheme PrebidMobileTests \
  -sdk iphonesimulator \
  -destination 'platform=iOS Simulator,name=iPhone-16-Pro-PrebidMobile,OS=latest' \
  build-for-testing

# Step 2 — run (repeat as needed without rebuilding)
xcodebuild \
  -workspace PrebidMobile.xcworkspace \
  -scheme PrebidMobileTests \
  -sdk iphonesimulator \
  -destination 'platform=iOS Simulator,name=iPhone-16-Pro-PrebidMobile,OS=latest' \
  -only-testing PrebidMobileTests/TargetingTests \
  test-without-building
```

See `agents/xcodebuild/SKILL.md` for a guided single-class run, and for the post-migration
build-error checklist.

### Setup

```bash
pod install --repo-update
```

Open `PrebidMobile.xcworkspace` (not `.xcodeproj`) for development.

## Architecture

### Workspace Structure

The workspace (`PrebidMobile.xcworkspace`) contains three projects:
- `PrebidMobile.xcodeproj` — core SDK + unit tests
- `EventHandlers/EventHandlers.xcodeproj` — ad network adapters
- `Example/PrebidDemo/PrebidDemo.xcodeproj` — demo apps

### Core SDK (`PrebidMobile/`)

Split between Swift and Objective-C layers:

**`PrebidMobile/Swift/`** — Public-facing Swift API:
- `Objc/Prebid.swift`, `Objc/Targeting.swift` — SDK configuration singleton and user targeting
- `Swift/AdUnits/` — `BannerAdUnit`, `InterstitialAdUnit`, `RewardedVideoAdUnit`, `InstreamVideoAdUnit`, `MultiformatAdUnit`
- `Swift/AdUnits/Native/` — Native ad unit types
- `Swift/ConfigurationAndTargeting/` — `AdUnitConfig`, targeting parameters
- `Swift/CacheManagement/` — bid caching layer
- `Swift/Host.swift` — Prebid Server endpoint configuration
- `Swift/Global.swift` — shared error types

**`PrebidMobile/Objc/`** — Internal Objective-C rendering engine:
- `PrebidMobileRendering/ORTB/` — OpenRTB bid request object model (`PBMORTBBidRequest`, `PBMORTBImp`, etc.)
- `PrebidMobileRendering/Networking/` — HTTP layer, URL building, impression tracking

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prebid/prebid-mobile-ios](https://github.com/prebid/prebid-mobile-ios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
