---
trigger: always_on
description: Handles plugin deferral first (see below), then hops to the isolation queue. It runs a full
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

This is the **Branch iOS SDK** for deep linking and attribution. Production code is a single
Objective-C library, `BranchSDK`, shipped for **iOS 12+ and tvOS 12+** via SPM, CocoaPods,
Carthage, and prebuilt XCFrameworks. There is no Swift in the shipped product on this branch —
everything under `Sources/BranchSDK/` is `.h`/`.m`.

> The `4.0.0-beta.0` line is a substantially different architecture (NSOperationQueue-based
> request queue, `/v3` endpoints). Do not apply this file's architecture notes to that branch.

## Products and distribution

One source tree, several delivery vehicles — all built from `Sources/BranchSDK/**/*.{h,m}`:

- **SPM** — `Package.swift`, target `BranchSDK`, `publicHeadersPath: BranchSDK/Public/`,
  private headers reachable only via the `BranchSDK/Private` header search path.
- **CocoaPods** — `BranchSDK.podspec`. tvOS **excludes** `BNCContentDiscoveryManager`,
  `BNCUserAgentCollector`, and `BNCSpotlightService`.
- **XCFramework** — `BranchSDK.xcodeproj` schemes `xcframework`, `xcframework-noidfa`,
  `static-xcframework`, driven by `scripts/build_xcframework*.sh` / `prep_*`.
- **IDFA-free variants** — the `-noidfa` scripts are the _same_ sources compiled with
  `GCC_PREPROCESSOR_DEFINITIONS=BRANCH_EXCLUDE_IDFA_CODE=1`. Any new AdSupport/IDFA touch
  must be guarded by that macro or the no-IDFA build breaks at App Store review time,
  not at compile time.

Version lives in **four** places, all rewritten by `scripts/version.sh`: the `version=` literal in
`scripts/version.sh` itself (the value the script reads back, so effectively the source of truth),
`BNC_SDK_VERSION` in `Sources/BranchSDK/BNCConfig.m`, `s.version` in `BranchSDK.podspec`, and
`MARKETING_VERSION` in `BranchSDK.xcodeproj/project.pbxproj` (6 build configurations). All four
currently agree at `3.14.2`. Never hand-edit any of them.

```bash
./scripts/version.sh        # print current version
./scripts/version.sh -i     # increment patch and update all four
./scripts/version.sh -u     # update the files to the current version, no increment
```

Do **not** reach for `bundle exec fastlane version_bump` or the `version-bump.yml` workflow:
`fastlane/lib/helper/version_helper.rb` still patches `carthage-files/`, `Branch-SDK/BNCConfig.m`,
`Branch.podspec` and `Branch-TestBed/Framework-Info.plist` — none of which exist on this branch.

## Source layout

Everything is flat under `Sources/BranchSDK/`, with headers split by visibility:

- **`Sources/BranchSDK/*.m`** — all implementations (72 files).
- **`Sources/BranchSDK/Public/`** — the public header surface. Adding a header here is a
  **public API change**; adding one to `Private/` is not.
- **`Sources/BranchSDK/Private/`** — internal headers (`BNCServerAPI.h`, `BNCRequestFactory.h`,
  `BNCPreferenceHelper` internals, request subclasses, …).
- **`Sources/Resources/`** — `PrivacyInfo.xcprivacy` (the privacy manifest), umbrella header,
  modulemap.

Naming convention is load-bearing: `BNC*` = internal/support type, `Branch*` = public-facing
type or a server request. `BNCServerRequest` subclasses are all named `Branch<Thing>Request`.

## Where to make changes

| Task                                                    | Start here                                                                                                                                                                                                                                                                                                                                                              |
| ------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Session/init flow, init status, deep-link callbacks     | `Branch.m` — `initUserSessionAndCallCallback:`, `initializeSessionAndCallCallback:`, `handleInitSuccessAndCallCallback:`, `handleInitFailure:`                                                                                                                                                                                                                          |
| Universal link / scheme / push / user-activity entry    | `Branch.m` — `handleDeepLink:sceneIdentifier:`, `handleSchemeDeepLink_private:`, `handleUniversalDeepLink_private:`, `continueUserActivity:`, `handlePushNotification:`                                                                                                                                                                                                 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BranchMetrics/ios-branch-deep-linking-attribution](https://github.com/BranchMetrics/ios-branch-deep-linking-attribution) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
