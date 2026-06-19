---
trigger: always_on
description: Use when the user wants to automate capture of marketing screenshots for a SwiftUI iOS app across multiple locales, devices, or appearances. Covers full-screen shots, isolated element renders (carousel cards, widgets), and reproducible output naming. Triggers on marketing screenshots, locale screenshots, widget renders, App Store assets, fastlane-alternative, simctl screenshots.
---


# iOS Marketing Capture

## Overview

Automate reproducible marketing screenshot capture for a SwiftUI iOS app across multiple locales, with two parallel output streams:

1. **Full-screen captures** — every marketing-relevant screen, with deterministic seeded data, real status bar / safe-area chrome
2. **Element captures** — isolated renders of specific components (cards, widgets, charts) at any scale, with natural background inside rounded corners and transparency outside

This skill is the **capture** step. If the user also wants Apple-style marketing pages composited around the shots (device mockups, headlines, gradients), combine with the `app-store-screenshots` skill as a post-processing step.

## Core Approach

**In-app capture mode**, not XCUITest. This is a hard decision that trades off against Fastlane snapshot / XCUITest conventions, and it wins for almost every real project.

Why in-app over XCUITest:

- **No new test target.** Adding a UI test target to an existing Xcode project is fragile pbxproj surgery. Many projects have zero test targets and no xcodegen — adding one by hand is error-prone.
- **Faster iteration.** A UI test takes 30s+ to launch per run. In-app capture is just a relaunch of the installed binary.
- **No `xcodebuild test`.** The whole flow is `xcodebuild build` once, then `simctl launch` per locale. No test-bundle overhead.
- **Access to real app state.** You can call ViewModels, SwiftData, ImageRenderer, and `UIWindow.drawHierarchy` directly. XCUITest can only tap and read accessibility elements.
- **Element renders need in-process anyway.** `ImageRenderer` on widget views or isolated components must run inside the app process — there's no XCUITest equivalent.

How it works:

1. A DEBUG-only `MarketingCapture.swift` file lives in the main app target
2. When launched with `-MarketingCapture 1`, the app seeds data, then a coordinator walks a list of `CaptureStep`s — each step navigates, waits for settle, snapshots, and cleans up
3. PNGs are written to the app's sandbox `Documents/marketing/<locale>/` directory
4. A shell script builds once, installs, then loops locales by relaunching with `-AppleLanguages (xx) -AppleLocale xx`, pulling files out via `simctl get_app_container`

## Process

Work through these steps in order. Do not skip ahead.

### Step 1: Gather requirements

Ask the user these questions **one at a time** (do not batch them — each answer can invalidate later questions):

1. **Screens to capture** — "Which screens do you want? Give me the navigation path or the tab name for each." Get a concrete list, not "the main flows".
2. **Isolated elements** — "Any components you want rendered independently with transparent backgrounds? (carousel cards, widgets, hero tiles, charts, etc.)"
3. **Locales** — "Which locales? (a) all locales in your `Localizable.xcstrings`, (b) an App Store subset I'll specify, or (c) let me give you an explicit list." If (a), grep the `.xcstrings` file for locale codes:
   ```bash
   python3 -c "import json; d=json.load(open('<path>/Localizable.xcstrings')); langs=set(); [langs.update(v.get('localizations',{}).keys()) for v in d['strings'].values()]; print(sorted(langs))"
   ```
4. **Device** — "Which simulator? (6.1\" iPhone 17 recommended for iOS 26 design features)" — verify the device is available via `xcrun simctl list devices available`.
5. **Appearance** — "Light only, dark only, or both?"
6. **Seed data** — "How is demo data populated today? (a) fresh install seeds it automatically, (b) there's a debug 'Load Demo Data' button, (c) you add it manually, (d) no demo data exists yet." Then: "Is the existing data exhaustive enough that every screen you listed looks populated for marketing? Audit it with the user."

### Step 2: Exploration

Before writing any code, explore the codebase enough to answer:

- Does the project use **Xcode synchronized folder groups** (Xcode 16+, `PBXFileSystemSynchronizedRootGroup`)? If yes, new files auto-include in their target — no pbxproj edits needed. Check with `grep -c PBXFileSystemSynchronized <proj>.xcodeproj/project.pbxproj`.
- **What is the root navigation pattern?**
  - `TabView(selection:)` — most common. You need: the `@State selectedTab` binding, tab indices, and which tabs have nested `NavigationStack`.
  - `NavigationStack` (single stack with a router) — you need: the path binding or router object, plus the set of `NavigationLink(value:)` / `.navigationDestination` types.
  - `NavigationSplitView` — you need: the sidebar selection binding, detail column's navigation state.
  - Custom coordinator / UIKit host — you need: the coordinator's `navigate(to:)` method or equivalent.
- How are **deep links** routed? Find the `onOpenURL` handler and the enum/switch that maps URLs to navigation state.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ParthJadhav/ios-marketing-capture](https://github.com/ParthJadhav/ios-marketing-capture) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
