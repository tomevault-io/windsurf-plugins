---
trigger: always_on
description: BG3 Mac Mod Manager is a macOS SwiftUI application for managing Baldur's Gate 3 mods. It uses Swift Package Manager (SPM) with swift-tools-version 5.9, targeting macOS 13+. Current release: **v1.4.0**.
---

# CLAUDE.md

## Project Overview

BG3 Mac Mod Manager is a macOS SwiftUI application for managing Baldur's Gate 3 mods. It uses Swift Package Manager (SPM) with swift-tools-version 5.9, targeting macOS 13+. Current release: **v1.4.0**.

## Build Environment

**In Claude Code environment, do not run `swift build`, `swift test`, or any Swift toolchain commands** — Swift is not installed in this environment. Verify correctness by reading code and checking that changes follow existing patterns. Note: the CI/CD release workflow does run tests before release.

## Project Structure

```
Sources/BG3MacModManager/
  App/          - App entry point (BG3MacModManagerApp), AppState
  Models/       - Data models (ModModel, ModCategory, NexusUpdateInfo, etc.)
  Services/     - Business logic (ModDiscoveryService, GameLaunchService, ModNotesService, NexusAPIService, NexusURLImportService, ArchiveService, ModValidationService, etc.)
  Utilities/    - Helpers (FileLocations, DesignTokens, etc.)
  Views/        - SwiftUI views (ContentView, ModListView, ModFilePickerView, NexusURLImportView, etc.)
Tests/BG3MacModManagerTests/
  TestHelpers.swift            - Factory functions (makeModInfo, makeDependency, makeSEStatus)
  Version64Tests.swift         - Version64 model tests
  ModInfoTests.swift           - ModInfo model, factories, MetadataSource, Constants
  DataBinaryReadingTests.swift - Data extension binary reading (UInt16/32/64, Int64)
  ModCategoryTests.swift       - ModCategory enum ordering, Codable, CaseIterable
  TextExportServiceTests.swift - CSV/Markdown/plain text export
  CategoryInferenceServiceTests.swift - Tag/name heuristics, overrides
  ModValidationServiceTests.swift     - All 10 validation checks, topological sort
  LoadOrderImportServiceTests.swift   - BG3MM JSON & LSX import parsing
  PakReaderTests.swift                - Binary format errors, CompressionType
  ModNotesServiceTests.swift          - Per-mod notes persistence
  NexusURLImportServiceTests.swift    - CSV/JSON/TXT import parsing, mod matching
  NexusAPIServiceTests.swift          - Mod ID extraction, update result logic, cache encoding
```

## Key Patterns

- **Icons**: SF Symbols only — no custom image assets
- **Button styles**: `.borderedProminent` for primary actions, `.bordered` for secondary, `.plain` for icon-only
- **Toolbar**: Standard macOS toolbar in ContentView; in-content action bars in views like ModListView for prominent buttons
- **All buttons** should have `.help()` tooltips
- **Help & tooltips**: Any changes or new features must update `HelpView.swift` documentation and ensure all new buttons/controls have `.help()` tooltips
- **State**: Single `AppState` (ObservableObject) passed via `.environmentObject()`
- **Async**: Use `Task { await ... }` in button actions for async AppState methods
- **Design tokens**: All semantic colors and spacing constants are defined in `DesignTokens.swift` — use these instead of inline opacity/color values. Severity colors are accessed via `ModWarning.Severity.color` and `.backgroundColor`
- **Animations**: Use `.spring(response: 0.3, dampingFraction: 0.8)` for list changes; `.easeInOut(duration: 0.2)` for transitions. Use `if #available(macOS 14, *)` guard for `.symbolEffect`

## Workflow

**Ask clarifying questions when uncertain.** If a prompt is ambiguous, has multiple possible interpretations, or you're unsure about the intended UX or behavior, ask the user before implementing. It is always better to clarify upfront than to implement the wrong thing and iterate.

**Always update CLAUDE.md when finishing updates.** After implementing a backlog item or making significant changes, mark the item as done in the Backlog section and update any affected documentation (project structure, key patterns, etc.) before committing.

**Before every `git push`, ask the user what version number to use** and update `CFBundleShortVersionString` in `Sources/BG3MacModManager/Info.plist` accordingly. Do not push without confirming the version.

## Dependencies

- [ZIPFoundation](https://github.com/weichsel/ZIPFoundation) (>=0.9.19) — ZIP archive handling

## Backlog

Prioritized feature and UX improvements organized by tier. Each item includes a scope tag (S/M/L) and the key files that need changes.

### Tier 1: Quick Wins

| # | Title | Scope | Status | Description | Key Files |
|---|-------|-------|--------|-------------|-----------|
| 1.1 | Unsaved Changes Indicator | S | **Done** | Track dirty state when load order is mutated; show indicator on Save button. Prevents silently losing work. | `AppState.swift`, `ModListView.swift` |
| 1.2 | Sidebar Mod Count Badges | S | **Done** | `.badge()` on sidebar items: warning count on Mods, profile count, backup count. | `ContentView.swift` |
| 1.3 | Keyboard Shortcuts | S | **Done** | Cmd+Shift+E (export ZIP), Cmd+Delete (deactivate selected), Cmd+Shift+G (launch game). Updated HelpView. | `BG3MacModManagerApp.swift`, `HelpView.swift` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ShaiLaric/BG3MacModManager](https://github.com/ShaiLaric/BG3MacModManager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
