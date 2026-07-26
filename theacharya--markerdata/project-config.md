---
trigger: always_on
description: This repository contains **Marker Data**, a macOS Swift/SwiftUI app that extracts Final Cut Pro marker metadata (via `MarkersExtractor`), optionally renders images/palettes, and can upload results to Notion/Airtable via bundled CLIs. It also ships a **Final Cut Pro Workflow Extension** and a **Share Destination** integration.
---

# AGENT.md

## Purpose
This repository contains **Marker Data**, a macOS Swift/SwiftUI app that extracts Final Cut Pro marker metadata (via `MarkersExtractor`), optionally renders images/palettes, and can upload results to Notion/Airtable via bundled CLIs. It also ships a **Final Cut Pro Workflow Extension** and a **Share Destination** integration.

This `AGENT.md` is guidance for humans and AI agents working in this repo: how to build, where to look, what to avoid, and how changes should be made.

For deeper module/data-flow detail, see **`ARCHITECTURE.md`**. For short agent guardrails, see **`.cursorrules`**.

## What you’re working on

| Area | Path |
|------|------|
| Main app (SwiftUI) | `Source/Marker Data/Marker Data/` |
| Workflow Extension | `Source/Marker Data/Workflow Extension/` |
| Uninstaller (SwiftUI) | `Source/Marker Data/Marker Data Uninstaller/` — target **Uninstall Marker Data**; product `Uninstall Marker Data.app`; display name “Marker Data Uninstaller”; bundle ID `co.theacharya.MarkerData.Uninstaller` |
| Share Destination install UI | `Source/Marker Data/Marker Data/FCP Share Destination/Install View/` |
| Share Destination scripting (Obj‑C) | `Source/Marker Data/Marker Data/FCP Share Destination/Objective-C Code/` |
| Bundled helper CLIs (opaque) | `Source/Marker Data/Marker Data/Resources/airlift`, `.../csv2notion_neo` |
| Distribution / DMG / Sparkle | `Distribution/` |
| CI | `.github/workflows/` |

**Xcode project:** `Source/Marker Data/Marker Data.xcodeproj`  
**Scheme:** **Marker Data** builds the main app (embeds Workflow Extension) and the Uninstall Marker Data target.

## Key entry points (start here)

| Concern | File |
|---------|------|
| `@main` app | `Source/Marker Data/Marker Data/Marker_DataApp.swift` — constructs `SettingsContainer`, `DatabaseManager`, `ExtractionModel`, `QueueModel`; menu commands; Failed Tasks + Pagemaker windows |
| AppKit / Sparkle delegate | `Source/Marker Data/Marker Data/ApplicationDelegate.swift` |
| Sidebar navigation | `Source/Marker Data/Marker Data/Views/Main/ContentView.swift` (`MainViews` enum) |
| Extract UI | `Source/Marker Data/Marker Data/Views/Main/ExtractView.swift` |
| Extraction orchestration | `Source/Marker Data/Marker Data/Models/Extract/Extraction Model/ExtractionModel.swift` |
| External handoffs (open / Workflow Extension) | `.../ExtractionModel_EventHandlers.swift` |
| Queue scan/upload | `Source/Marker Data/Marker Data/Models/Queue/QueueModel.swift` |
| Database uploads | `Source/Marker Data/Marker Data/Models/Extract/DatabaseUploader.swift` |
| Settings schema | `Source/Marker Data/Marker Data/Models/Settings/SettingsStore.swift` (`static let version` — currently **8**) |
| Settings container / configs | `Source/Marker Data/Marker Data/Models/Settings/SettingsContainer.swift` |
| Settings migrations | `Source/Marker Data/Marker Data/Models/Settings/SettingsVersioningManager.swift` |
| Canonical disk paths | `Source/Marker Data/Marker Data/Utilities/Extensions/URLExtension.swift` |
| Notification names | `Source/Marker Data/Marker Data/Utilities/Extensions/NotificationNameExtension.swift` |
| Alert dialog icon helper | `Source/Marker Data/Marker Data/Views/Extensions/DialogIcon.swift` (`.appDialogIcon()`) |

## Build & run (local)
1. Open `Source/Marker Data/Marker Data.xcodeproj`.
2. Select the **Marker Data** scheme.
3. Build/run for **Apple Silicon** (`arm64`). Do not target Intel.

CI installs the Workflow Extensions SDK from `SDK/Workflow_Extensions_1.0.3.dmg` before `xcodebuild`. Locally you need the same SDK installed under `/Library/Developer/SDKs/WorkflowExtensionSDK.sdk` to build the extension target.

**SwiftFormat** (courtesy, not CI-enforced): from repo root, `swiftformat .` — see `CONTRIBUTING.md`.

## CI / release basics

| Fact | Value |
|------|--------|
| Runner | `macos-26` |
| Xcode (workflows) | **`Xcode_26.6.0`** (`sudo xcode-select -s /Applications/Xcode_26.6.0.app/...`) |
| Architecture | `arch=arm64`, `EXCLUDED_ARCHS=x86_64` |
| PR/push build | `.github/workflows/build.yml` |
| Test (notarized) builds | `test_build.yml`, `test_build_debug.yml` |
| Full release + Sparkle appcast | `release_github.yml` |
| Release without appcast | `release_github_non-appcast.yml` |
| Refresh bundled CLIs / Pagemaker | `update_airlift_binary.yml`, `update_csv2notion_neo_binary.yml`, `update_pagemaker.yml` |

Release flow (high level):
1. Build **Marker Data** scheme (main app + Uninstall Marker Data).
2. Copy `Marker Data.app` and `Uninstall Marker Data.app` into `latest-build/`.
3. Codesign: Workflow Extension → Sparkle framework/XPC helpers → main app → Uninstaller.
4. Notarize; package DMG with `appdmg` + `Distribution/dmg-builds/build-marker-data-dmg.json`.
5. Sparkle: feed is `appcast.xml`; updated by `Distribution/dmg-builds/sparkle/generate_appcast_script.py`.

Shipping philosophy: distribute the **Derived Data Release `.app`**, not an Archive (debuggability / reproducibility — see `CONTRIBUTING.md`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheAcharya/MarkerData](https://github.com/TheAcharya/MarkerData) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
