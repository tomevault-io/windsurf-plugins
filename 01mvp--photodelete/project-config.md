---
trigger: always_on
description: This file provides guidance when working in this repository.
---

# AGENTS.md

This file provides guidance when working in this repository.

The project is licensed under AGPL-3.0. Do not relicense it, and do not describe closed-source forks as allowed.

## Repository Rules

- Do not put internal instructions, agent notes, TODO process text, or hidden implementation guidance into any user-facing UI, website copy, App Store copy, screenshots, or localized strings.
- When the work can be split cleanly, use independent subagents for non-overlapping read-only research or disjoint file scopes, then integrate and verify in the main thread.
- Treat the worktree as shared. Check `git status --short` before edits, do not revert user changes, and keep staging/commits narrowly scoped when asked to commit.
- Public UI text must be concise, user-facing, and localized through `L10n` / `Localizable.xcstrings`. Avoid leaking raw technical failures unless they are only for diagnostics.
- Core cleanup remains free. Supporter features are a one-time StoreKit unlock; do not describe the app as subscription-only or imply that basic cleanup requires payment.

## Project Overview

PhotoDelete is an iOS 16+ SwiftUI app for organizing and cleaning a real Photos library. The app uses swipe gestures, a safe candidate library, batch confirmation, album management, local cleanup history, and an optional StoreKit supporter unlock for advanced statistics and cleanup queues.

Privacy positioning is part of the product: no account is required, photos are processed on-device, and the app does not upload photos, videos, library contents, or cleanup decisions.

## Development Commands

### Open In Xcode

```bash
cd IOSAPP
open PhotoDelete.xcodeproj
```

Build and run through Xcode. Use a simulator for quick UI work and a real iPhone for Photos framework, iCloud Photos, limited library, deletion, favorite, and album write validation.

### Build From CLI

```bash
SIMULATOR_DESTINATION="$(scripts/resolve-ios-simulator-destination.sh)"
xcodebuild -project IOSAPP/PhotoDelete.xcodeproj -scheme PhotoDelete \
  -destination "$SIMULATOR_DESTINATION" \
  -derivedDataPath IOSAPP/DerivedData \
  clean build
```

### CI-Style Unit Test Flow

```bash
SIMULATOR_DESTINATION="$(scripts/resolve-ios-simulator-destination.sh)"
xcodebuild build-for-testing \
  -project IOSAPP/PhotoDelete.xcodeproj \
  -scheme PhotoDelete \
  -destination "$SIMULATOR_DESTINATION" \
  -derivedDataPath IOSAPP/DerivedData \
  CODE_SIGNING_ALLOWED=NO

xcodebuild test-without-building \
  -project IOSAPP/PhotoDelete.xcodeproj \
  -scheme PhotoDelete \
  -destination "$SIMULATOR_DESTINATION" \
  -derivedDataPath IOSAPP/DerivedData \
  -only-testing:PhotoDeleteTests \
  CODE_SIGNING_ALLOWED=NO
```

The GitHub Actions workflow in `.github/workflows/ios-ci.yml` follows this pattern on `macos-15` with Xcode 16.4.

### Full Simulator Test

```bash
SIMULATOR_DESTINATION="$(scripts/resolve-ios-simulator-destination.sh)"
xcodebuild test \
  -project IOSAPP/PhotoDelete.xcodeproj \
  -scheme PhotoDelete \
  -destination "$SIMULATOR_DESTINATION" \
  -derivedDataPath IOSAPP/DerivedData
```

UI tests are smoke tests. They do not replace real-device Photos validation.

### Cleanup

```bash
rm -rf IOSAPP/DerivedData
xcodebuild clean -project IOSAPP/PhotoDelete.xcodeproj -scheme PhotoDelete
```

### TestFlight Release

- TestFlight build numbers must use UTC+8 time in `yyyyMMddHHmm` format, matching Beijing/Shanghai local time. Do not generate release build numbers from UTC time.
- Before uploading, make sure the new `CFBundleVersion` is numerically greater than the highest build already visible in App Store Connect/TestFlight; otherwise testers may not receive it as an update even if the upload succeeds.
- Do not manually bump `MARKETING_VERSION` for every TestFlight upload. Reuse the current version while the pre-release train accepts new builds; if App Store Connect rejects the upload because the train is closed or `CFBundleShortVersionString` must be higher than the approved version, let `scripts/release-testflight.sh` auto-increment the last marketing-version component once, write it back to the Xcode project, and retry with a fresh UTC+8 build number.
- The release script accepts an explicit override:

```bash
BUILD_NUMBER=202606111630 scripts/release-testflight.sh
```

`scripts/release-testflight.sh` runs tests unless `SKIP_TESTS=1`, checks that App Icon PNGs do not contain alpha channels, archives, exports, uploads to App Store Connect/TestFlight, and handles one automatic marketing-version retry when Apple closes the current train.

## Architecture

### App Shell

- `PhotoDeleteApp.swift`: App entry point, UI-test defaults, gesture preference migration, selected app language, and selected appearance.
- `ContentView.swift`: First-run onboarding and root switch into the main app.
- `MainTabView.swift`: Four tabs: Organize, Albums, Advanced, Settings.

### Core Data And Services

- `Models.swift`: Photo categories, gesture actions/presets, review modes, time groups, album models, app appearance, and advanced cleanup models.
- `DataManager.swift`: Central observable state, candidate libraries, reviewed asset IDs, time groups, album lists, batch operations, advanced summaries, and cleanup statistics coordination.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [01MVP/PhotoDelete](https://github.com/01MVP/PhotoDelete) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
