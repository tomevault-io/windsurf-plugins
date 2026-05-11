---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
# Open in Xcode
xed .

# Build DMG (cleans derived data, builds Release, creates DMG in project root)
./build-dmg.sh

# Build from command line (Debug) — must override team ID
xcodebuild -project "Nano Banana Helper.xcodeproj" -scheme "Nano Banana Helper" -configuration Debug DEVELOPMENT_TEAM=46BZ85ALNS

# Run all tests (deployment target override needed if host macOS < SDK)
xcodebuild -project "Nano Banana Helper.xcodeproj" -scheme "Nano Banana Helper" -destination 'platform=macOS' test MACOSX_DEPLOYMENT_TARGET=26.2

# Run a single unit test (Swift Testing framework) — target ID uses underscores
xcodebuild -project "Nano Banana Helper.xcodeproj" -scheme "Nano Banana Helper" -destination 'platform=macOS' test MACOSX_DEPLOYMENT_TARGET=26.2 -only-testing:Nano_Banana_HelperTests/ClassName/testName

# Run a single UI test (XCTest framework)
xcodebuild -project "Nano Banana Helper.xcodeproj" -scheme "Nano Banana Helper" -destination 'platform=macOS' test -only-testing:Nano-Banana-HelperUITests/ClassName/testName
```

**Always build a DMG after any code change.** Run `./build-dmg.sh` at the end of every task. The DMG is stored at `Nano Banana Helper.dmg` in the project root.

## Architecture Overview

MVVM with SwiftUI Observation framework. Three-layer structure:

```
Views/           → SwiftUI views, consume @Environment objects
Services/        → Business logic, API clients, coordination
Models/          → Data structures, @Observable state managers
```

**No external dependencies** — pure Apple SDK (SwiftUI, Foundation, Observation, UserNotifications, AppKit, UniformTypeIdentifiers).

**Sandbox**: App runs with App Sandbox enabled. Entitlements are set in build settings (no `.entitlements` file): outgoing/incoming network connections, user-selected file access (read-write), Downloads folder (read-write), Hardened Runtime.

**File system sync**: Xcode uses `PBXFileSystemSynchronizedRootGroup` — the file system IS the source of truth. Create files in the correct directory; no need to manually add them to the Xcode project.

### Key State Objects

| Object | File | Role |
|--------|------|------|
| `BatchOrchestrator` | Services/ | Job queue, concurrent execution, polling. Created at App level, injected via `.environment()` |
| `ProjectManager` | Services/ | Project CRUD, cost tracking, persistence. Created as `@State` in `MainLayoutView` |
| `BatchStagingManager` | Models/ | Staged files, batch configuration (prompt, aspect ratio, size). Passed as `@Bindable`. `generationMode` (`.image` or `.text`) controls API call path — `.image` edits existing images, `.text` generates from scratch |
| `PromptLibrary` | Models/ | Saved prompt templates (user/system types) |
| `HistoryManager` | Services/ | Global history aggregation |
| `LogManager` | Models/Models.swift | In-memory session logger (`@Observable @MainActor` singleton) |
| `AppConfig` | Services/NanoBananaService.swift | API key, model name. `@MainActor` struct persisted to `config.json`. Defined alongside request/response/error types (`ImageEditRequest`, `ImageEditResponse`, `BatchJobInfo`, `NanoBananaError`) |
| `TokenUsage` | Models/BillingModels.swift | `nonisolated` struct with prompt/candidates/total token counts. Decoded from API `usageMetadata` |
| `CostSummary` | Models/Models.swift | Extended with `totalTokens`, `inputTokens`, `outputTokens`, `byModel`. Custom `Codable` for backward compat (new fields default to 0/empty) |
| `AppPaths` | Models/AppPaths.swift | Centralized path management, security-scoped bookmark helpers |
| `Project` | Models/Models.swift | `@Observable class` — domain model grouping batch jobs. Properties: id, name, outputDirectory, totalCost, presets |
| `HistoryEntry` | Models/Models.swift | Core data type for a completed image edit with token usage, model name, cost metadata |
| `ImageTask` | Models/Models.swift | `@Observable class` — single image task within a batch, multi-input support |
| `BatchJob` | Models/Models.swift | `@Observable class` — batch container with `isTextMode` flag |
| `JobPhase` | Models/Models.swift | Enum: `.pending`, `.submitting`, `.polling`, `.reconnecting`, `.downloading`, `.completed`, `.failed` |
| `ImageSize` | Models/Models.swift | Enum: `512`, `1K`, `2K`, `4K` with `standardCost`/`batchCost`/`calculateCost()` |
| `AspectRatio` | Models/AspectRatio.swift | Supported output ratios with categories (auto, square, landscape, portrait) |
| `GenerationMode` | Models/BatchStagingManager.swift | Enum: `.image` (edit existing) or `.text` (generate from scratch) |
| `Constants` | Models/Constants.swift | App-wide constants (`maxTextImageVariations = 4`) |

### View Hierarchy

```
Nano_Banana_HelperApp (@main) → ContentView → MainLayoutView
  NavigationSplitView: SidebarView | WorkbenchView (Staging/Results/History) + InspectorView
  ProgressQueueView, BottomDockView
  Sheets: SettingsView, NewProjectSheet, CostReportView, UsageDashboardView (inside Settings)
```

All state objects besides `BatchOrchestrator` are created and callbacks wired in `MainLayoutView.onAppear`.

### Data Flow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joshmac007/Nano-Banana-Helper](https://github.com/joshmac007/Nano-Banana-Helper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
