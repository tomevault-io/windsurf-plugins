---
trigger: always_on
description: This repository is a native iOS/iPadOS Typst editor built with SwiftUI, SwiftData, and a Rust FFI bridge.
---

# AGENTS.md

This repository is a native iOS/iPadOS Typst editor built with SwiftUI, SwiftData, and a Rust FFI bridge.

## Project Overview

- App target: `InkPond`
- Minimum deployment target: iOS/iPadOS 17.0
- Optional iOS 26-only UI enhancements are used behind availability checks
- Data model: `InkPondDocument` (`@Model`)
- Typst compilation: Rust static library packaged as `Frameworks/typst_ios.xcframework`
- Pinned Typst version: `0.14.2` (see `rust-ffi/Cargo.toml`)

## Build Setup

The app depends on a generated Rust xcframework. Build it before the first Xcode build or after Rust changes:

```bash
cd rust-ffi
./build-ios.sh
cd ..
```

The xcframework output is generated locally into `Frameworks/typst_ios.xcframework` and is not committed. The build script produces three targets: `aarch64-apple-ios` (device), `aarch64-apple-ios-sim` (M-series simulator), and `x86_64-apple-ios` (Intel simulator).

## Build & Test Commands

```bash
# Debug build for simulator
xcodebuild -project InkPond.xcodeproj -scheme InkPond -configuration Debug -destination 'generic/platform=iOS Simulator' build

# Release archive for device
xcodebuild -project InkPond.xcodeproj -scheme InkPond -configuration Release -destination 'generic/platform=iOS' archive

# Unit tests
xcodebuild test -project InkPond.xcodeproj -scheme InkPond -only-testing:InkPondTests -destination 'platform=iOS Simulator,name=iPhone 17,OS=26.2'

# UI tests
xcodebuild test -project InkPond.xcodeproj -scheme InkPond -only-testing:InkPondUITests -destination 'platform=iOS Simulator,name=iPhone 17,OS=26.2'
```

If the named simulator is unavailable locally, inspect options first:

```bash
xcodebuild -showdestinations -project InkPond.xcodeproj -scheme InkPond
```

## Architecture

### App Shell
- `InkPond/InkPondApp.swift`: `@main` entry point; sets up SwiftData `ModelContainer` with `InkPondDocument` schema; initializes `SnippetStore`; runs startup cleanup (temp exports, font cache pruning)
- `InkPond/ContentView.swift`: `NavigationSplitView` shell; sidebar = document list, detail = editor; injects `ThemeManager`, `AppAppearanceManager`, `AppFontLibrary` into environment; onboarding gate

### Data Model
- `InkPond/Models/InkPondDocument.swift`: `@Model` with properties: `title`, `content`, `createdAt`, `modifiedAt`, `fontFileNames[]`, `projectID`, `entryFileName`, `imageInsertMode`, `imageDirectoryName`, `lastEditedFileName`, `lastCursorLocation`, `requiresInitialEntrySelection`, import option arrays

### Editor Layer
- `InkPond/Editor/TypstTextView.swift`: `UITextView` subclass forced to TextKit 1; integrates gutter, syntax highlighting, completion popup, auto-pairing, keyboard accessory, find/replace, rich paste handling (HTML→text, image extraction, remote URL fetch), keyboard avoidance
- `InkPond/Editor/SyntaxHighlighter.swift`: 15 ordered regex rules (bold/italic, numbers+units, math, code blocks, inline code, labels/refs, keywords, literals, functions, bools, hash-keywords, headings, strings, block comments, line comments); rainbow bracket pass (6-color cycle); bracket mismatch detection (red underline); error line highlighting; jump highlight animation
- `InkPond/Editor/CompletionEngine.swift`: context-aware completion for hash-prefix (~150 functions + keywords + snippets), parameters (after open paren), values (fonts, labels, image paths), at-prefix (@refs), angle-bracket labels; sources include font paths, BibTeX entries, external labels, image files
- `InkPond/Editor/AutoPairEngine.swift`: pairs `{}[]()""$$`; type-over, auto-close, auto-delete, quote/$ awareness, auto-indent on Enter (4-space indent in brackets, triple newline with dedent); all ops registered with undo manager
- `InkPond/Editor/SyncCoordinator.swift`: bidirectional editor↔preview sync; direction lock + 150ms cooldown to prevent feedback loops; editor→preview on tap/arrow, preview→editor on PDF tap
- `InkPond/Editor/EditorTheme.swift` + `ThemeManager.swift`: Mocha (forced dark), Latte (forced light), System (adaptive); Catppuccin color palette; persisted in UserDefaults
- `InkPond/Editor/Snippet.swift` + `SnippetLibrary.swift` + `SnippetStore.swift`: snippet model with `$0` cursor placeholder; built-in library + user custom; persisted as JSON in `Library/ApplicationSupport/Snippets.json`
- `InkPond/Editor/HighlightScheduler.swift`: debounced syntax highlighting scheduling
- `InkPond/Editor/LineNumberGutterView.swift`: gutter with theme-aware colors and error line markers
- `InkPond/Editor/KeyboardAccessoryView.swift`: accessory bar with photo import and snippet insertion buttons

### Compiler Layer
- `InkPond/Compiler/TypstBridge.swift`: Rust FFI wrapper; `compile()` → PDF data; `compileWithSourceMap()` → PDF + `SourceMap`; null-terminated UTF-8 source, font paths (bundled CJK + project + app), rootDir for local file resolution
- `InkPond/Compiler/TypstCompiler.swift`: `@Observable @MainActor`; debounced compilation (350ms, `.utility` priority) and immediate compilation (`.medium` priority, for export); 30s timeout; integrates `CompiledPreviewCacheStore`; exposes `pdfDocument`, `errorMessage`, `sourceMap`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lin0u0/InkPond](https://github.com/Lin0u0/InkPond) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
