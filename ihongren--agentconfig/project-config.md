---
trigger: always_on
description: A native macOS app (SwiftUI + AppKit) for managing AI coding agent configuration files and agent profiles (Codex, Claude Code).
---

# AGENTS.md

# AgentConfig — Project-specific

A native macOS app (SwiftUI + AppKit) for managing AI coding agent configuration files and agent profiles (Codex, Claude Code).

## Build

```
xcodebuild -project AgentConfig/AgentConfig.xcodeproj -scheme AgentConfig -configuration Debug build
```

## Project defaults

- `SWIFT_DEFAULT_ACTOR_ISOLATION = MainActor` — **all Swift code defaults to `@MainActor`**. Don't add `@MainActor` annotations, they're redundant.
- macOS deployment target: 14.0
- App Sandbox: disabled. Hardened Runtime: enabled.
- No Swift Package Manager dependencies.
- The Xcode project uses a filesystem-synchronized root group, so adding/removing Swift files under `AgentConfig/AgentConfig` is picked up by the target without manually editing `project.pbxproj` in ordinary cases.

## Tests

**There are no tests.** No XCTest target exists. When asked to add tests, create a test target in Xcode first.

## Architecture

**MVVM** with protocol-based service injection into ViewModels. `AppViewModel` is shared with sidebar/settings views via `environmentObject`; other ViewModels are passed explicitly through initializers.

```
Views (SwiftUI + AppKit via NSViewRepresentable)
  └─ ViewModels (@MainActor, ObservableObject, @Published)
      └─ Services (protocols, injected via init)
          └─ Models (plain structs/enums)
```

### Layout

`NavigationSplitView` with two columns: sidebar (`SidebarView` + `FileListView`) and detail pane. The detail pane switches between `EditorView` (for config files), `CodexProfileEditorView`, and `ClaudeProfileEditorView` based on sidebar selection. Responsive: sidebar auto-collapses when window width < 760pt.

### Key components

- **AgentConfigApp.swift** — `@main` entry point, `AppDelegate` (posts `appDidBecomeActive` for file-watcher refresh), `CommandCoordinator` (bridges editor callbacks to menu commands), `MainContentView`. Owns all `@StateObject` ViewModels: `AppViewModel`, `EditorViewModel`, `CodexProfileViewModel`, `ClaudeProfileViewModel`, `CommandCoordinator`.
- **ContentView.swift** — Placeholder view (not used in main flow).
- **AppViewModel** — central coordinator: owns AgentScanner, FileService, AppSettings. Handles file selection, custom paths, hide/show files. Defines `CategorySelection` enum (`.agent` / `.env`).
- **EditorViewModel** — file editing (load/save/undo/redo/search/JSON format), external change detection, source runner for shell files. Content is a `@Published` two-way binding to the NSTextView.
- **CodexProfileViewModel** — manages Codex profile selection, editing, persistence, and applying profiles to disk.
- **ClaudeProfileViewModel** — manages Claude Code profile selection, editing, persistence, and applying profiles to disk. Mirrors `CodexProfileViewModel` structure.
- **SidebarView** — shows environment files, known agent files, custom-added files, missing files, plus profile subsections under Codex and Claude Code agents.
- **FileListView** — middle column within the sidebar area; lists files for the selected category with creation buttons for missing files and unsaved-change indicators.
- **EditorView / CodeEditorView** — `NSTextView` wrapped via `NSViewRepresentable`. Syntax highlighting via `SyntaxHighlighter` (`NSTextStorageDelegate`, regex-based). Comment toggle via `CommentingTextView` (Cmd+/ for `#`, `//`, `%` prefixes based on `FileType`).
- **EditorToolbarView** — top toolbar showing filename (with unsaved indicator), JSON format button, and inline format-error location.
- **SearchBarView** — VSCode-style search bar with keyword highlighting, prev/next navigation, and case-sensitivity toggle.
- **AgentProfileEditorView** — generic, reusable profile editor driven by `AgentProfileEditorProfile` / `AgentProfileEditorField` value types. Used by both `CodexProfileEditorView` and `ClaudeProfileEditorView`.
- **AgentProfileCodeEditor** — resizable code editor cards for `AgentProfileEditorView`, with drag-to-resize handles and height persistence.
- **CodexProfileEditorView** — adapts `AgentProfileEditorView` for Codex profiles (config TOML, auth JSON, zshrc exports).
- **ClaudeProfileEditorView** — adapts `AgentProfileEditorView` for Claude Code profiles (settings JSON, claude.json, zshrc exports).
- **ProfileFieldHelpButton** — small `?` button that shows a popover with field help text.
- **CommentingTextView** — `NSTextView` subclass handling Cmd+/ comment toggling, dispatching to `FileType.lineCommentPrefix`.
- **SettingsView** — appearance (light/dark/system) and language (en/zh-Hans/system) settings.
- **AboutView** — app info, version, build number, author credits.
- **AgentScanner** — scans `~` for known agent config files using definitions from `AgentDefinitions` (2 enabled: Claude Code, Codex; 3 commented-out: Gemini CLI, OpenCode CLI, Qwen Code).
- **FileWatcher** — `DispatchSourceFileSystemObject` with 500ms debounce for external change detection.
- **AppSettings** — `UserDefaults` wrapper for appearance, language, customPaths, hiddenFiles, and per-category added file paths.
- **AppError** — `LocalizedError` enum: `fileReadFailed`, `fileWriteFailed`, `jsonFormatError`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iHongRen/AgentConfig](https://github.com/iHongRen/AgentConfig) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
