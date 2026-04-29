---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Clearly is a native macOS markdown editor built with SwiftUI. It's a document-based app (`DocumentGroup`) that opens/saves `.md` files, with two modes: a syntax-highlighted editor and a WKWebView-based preview. It also ships a QuickLook extension for previewing markdown files in Finder.

## Build & Run

The Xcode project is generated from `project.yml` using [XcodeGen](https://github.com/yonaskolb/XcodeGen):

```bash
xcodegen generate        # Regenerate .xcodeproj from project.yml
xcodebuild -scheme Clearly -configuration Debug build   # Build from CLI
```

Open in Xcode: `open Clearly.xcodeproj` (gitignored, so regenerate with xcodegen first).

- Deployment target: macOS 14.0
- Swift 5.9, Xcode 16+
- Dependencies: `cmark-gfm` (GFM markdown → HTML), `Sparkle` (auto-updates, direct distribution only), `GRDB` (SQLite + FTS5), `MCP` (Model Context Protocol SDK) via Swift Package Manager

## Architecture

**Three targets** defined in `project.yml`:

1. **Clearly** (main app) — document-based SwiftUI app
2. **ClearlyQuickLook** (app extension) — QLPreviewProvider for Finder previews
3. **ClearlyMCP** (command-line tool) — MCP server exposing vault index to AI agents. Shares `VaultIndex.swift`, `FileParser.swift`, `FileNode.swift`, `DiagnosticLog.swift` with the main app, plus `FrontmatterSupport.swift` from `Shared/`. Uses `init(locationURL:bundleIdentifier:)` to open the same SQLite index the sandboxed app creates. Exposes 3 tools: `search_notes` (FTS5 ranked search), `get_backlinks` (linked + unlinked mentions), `get_tags` (tag aggregation). Read-only index access via WAL mode.

**Shared code** lives in `Shared/` and is compiled into both targets:
- `MarkdownRenderer.swift` — wraps `cmark_gfm_markdown_to_html()` for GFM rendering. Post-processing pipeline (in order): math (`$...$` → KaTeX spans), highlight marks (`==text==` → `<mark>`), superscript/subscript, emoji shortcodes, callouts/admonitions (`[!TYPE]` blockquotes), TOC generation, table captions, code filename headers. All post-processing that touches inline syntax must use `protectCodeRegions()`/`restoreProtectedSegments()` to avoid transforming content inside `<pre>`/`<code>` tags.
- `PreviewCSS.swift` — CSS string used by both the in-app preview and the QuickLook extension
- `MathSupport.swift` / `MermaidSupport.swift` / `TableSupport.swift` — conditional JS injection for preview features. Each follows the same pattern: check if the HTML contains relevant content, return script HTML or empty string

**App code** in `Clearly/`:
- `ClearlyApp.swift` — App entry point. `DocumentGroup` with `MarkdownDocument`, menu commands for switching view modes (⌘1 Editor, ⌘2 Preview)
- `MarkdownDocument.swift` — `FileDocument` conformance for reading/writing markdown files
- `ContentView.swift` — Hosts the mode picker toolbar and switches between `EditorView` and `PreviewView`. Defines `ViewMode` enum and `FocusedValueKey` for menu commands
- `EditorView.swift` — `NSViewRepresentable` wrapping `NSTextView` with undo, find panel, and live syntax highlighting via `NSTextStorageDelegate`
- `MarkdownSyntaxHighlighter.swift` — Regex-based syntax highlighter applied to `NSTextStorage`. Handles headings, bold, italic, code blocks, links, blockquotes, lists, etc. Code blocks are matched first to prevent inner highlighting
- `PreviewView.swift` — `NSViewRepresentable` wrapping `WKWebView` that renders the full HTML preview
- `Theme.swift` — Centralized colors (dynamic light/dark via `NSColor(name:)`) and font/spacing constants

**Key pattern**: The editor uses AppKit (`NSTextView`) bridged to SwiftUI via `NSViewRepresentable`, not SwiftUI's `TextEditor`. This is intentional — it provides undo support, find panel, and `NSTextStorageDelegate`-based syntax highlighting.

**Threading rule for `FileNode.buildTree()`**: This method does recursive filesystem I/O and must never run on the main thread. Use `WorkspaceManager.loadTree(for:at:reindex:)` which dispatches to a background queue, guards against stale completions via a generation counter, and assigns the result on main. The same applies to any new code that calls `FileManager.contentsOfDirectory` over potentially large directory trees.

**Avoid `.inspector()` for panels that must look correct in fullscreen.** SwiftUI's `.inspector()` introduces an internal safe area gap at the top of the panel in fullscreen mode. The background and borders don't extend to the window edge, and there's no reliable way to fix it from the outside — painting ancestor NSViews, `.toolbarBackgroundVisibility(.hidden)`, ZStack backgrounds with `.ignoresSafeArea()`, and adding border subviews to AppKit containers were all tried and failed or caused dark mode / alignment regressions. The outline panel was converted from `.inspector()` to a plain `HStack` sibling with a manual 1px separator for this reason.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shpigford/clearly](https://github.com/Shpigford/clearly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
