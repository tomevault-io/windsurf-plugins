---
trigger: always_on
description: This guide explains how to integrate PierreDiffsSwift's full feature set into a consumer app, with specific focus on the inline annotation (code review) system.
---

# PierreDiffsSwift Integration Guide

This guide explains how to integrate PierreDiffsSwift's full feature set into a consumer app, with specific focus on the inline annotation (code review) system.

## Package Dependency

```swift
// Package.swift
.package(url: "https://github.com/jamesrochabrun/PierreDiffsSwift", from: "1.2.0")
```

## Core View: PierreDiffView

```swift
PierreDiffView(
    oldContent: String,                                          // Original file content
    newContent: String,                                          // Updated file content
    fileName: String,                                            // For syntax highlighting
    diffStyle: Binding<DiffStyle>,                               // .split or .unified
    overflowMode: Binding<OverflowMode>,                         // .scroll or .wrap
    annotations: [DiffAnnotation]? = nil,                        // Inline comments
    onLineClick: ((Int, String) -> Void)? = nil,                 // (lineNumber, side)
    onLineClickWithPosition: ((LineClickPosition, CGPoint) -> Void)? = nil,  // With screen position
    onLineSelectionChange: ((LineSelectionRange) -> Void)? = nil, // Multi-line drag
    onAnnotationClick: ((String, String, Int, CGPoint) -> Void)? = nil,      // (id, side, lineNumber, localPoint)
    onAnnotationDelete: ((String, String, Int) -> Void)? = nil,              // (id, side, lineNumber)
    onExpandRequest: (() -> Void)? = nil,
    onReady: (() -> Void)? = nil
)
```

All callbacks are optional. The view works with just content + fileName + bindings.

## Feature 1: Line Click with Position

Use `onLineClickWithPosition` to get a `CGPoint` for positioning SwiftUI overlays (editors, popovers, menus) relative to the WebView.

```swift
onLineClickWithPosition: { position, localPoint in
    // position.lineNumber: Int — the clicked line
    // position.side: String — "left", "right", or "unified"
    // localPoint: CGPoint — WebView-local coordinates, top-left origin (matches SwiftUI)
    editorOverlay.show(at: localPoint, lineNumber: position.lineNumber, side: position.side)
}
```

The `CGPoint` is computed from `NSEvent.mouseLocation` converted to WebView-local coordinates. It is accurate regardless of scroll position.

## Feature 2: Multi-Line Selection

Use `onLineSelectionChange` to detect when users drag across line numbers.

```swift
onLineSelectionChange: { selection in
    // selection.startLine: Int
    // selection.endLine: Int
    // selection.side: String — "left", "right", or "unified"
    editorOverlay.show(lineRange: selection.startLine...selection.endLine, side: selection.side)
}
```

## Feature 3: Inline Annotations (Code Review)

Annotations are inline comment blocks rendered inside the WebView below the target line. They support click-to-edit and click-to-delete interactions.

### Data Model

```swift
DiffAnnotation(
    side: .additions,        // .additions (right/new) or .deletions (left/old)
    lineNumber: 42,          // Line number to attach to
    metadata: AnnotationMetadata(
        id: "unique-id",     // Used for identification in callbacks (defaults to UUID)
        author: "You",       // Display name shown in annotation header
        body: "Comment text" // The comment body
    )
)
```

### Reactive Data Flow

PierreDiffsSwift is **stateless** regarding annotations. The consumer owns the comment state and passes `[DiffAnnotation]` as a prop. SwiftUI reactivity handles the rest.

```
Consumer State (@Observable)
    ↓ converts comments → [DiffAnnotation]
PierreDiffView(annotations: [...])
    ↓ updateNSView detects change
coordinator.setAnnotations([...])
    ↓ JS bridge
@pierre/diffs renders inline DOM (no full re-render)
```

When annotations change (add/edit/remove), just update your state. The annotations array changes, SwiftUI re-evaluates, and `updateNSView` calls `setAnnotations()` automatically.

### Callbacks

**`onAnnotationClick: (id, side, lineNumber, localPoint)`**
User clicked the annotation body. Use this to open an edit overlay at `localPoint`.

**`onAnnotationDelete: (id, side, lineNumber)`**
User clicked the X button on the annotation. Use this to remove the comment from your state. The annotation disappears reactively when the `annotations` array updates.

## Integration Pattern: Inline Code Review

This is the recommended pattern for building a PR-style inline review system. This is how a consumer like AgentHub should integrate.

### Step 1: Comment State Manager

Create an `@Observable` class that stores review comments and converts them to `[DiffAnnotation]`:

```swift
@Observable @MainActor
class ReviewCommentsState {
    // Store comments keyed by location for deduplication
    var comments: [String: ReviewComment] = [:]

    var orderedComments: [ReviewComment] {
        comments.values.sorted { $0.timestamp < $1.timestamp }
    }

    var commentsByFile: [String: [ReviewComment]] {
        Dictionary(grouping: orderedComments, by: \.filePath)
    }

    // Convert stored comments to DiffAnnotation array for a specific file
    func annotations(for filePath: String) -> [DiffAnnotation] {
        commentsByFile[filePath]?.map { comment in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jamesrochabrun/PierreDiffsSwift](https://github.com/jamesrochabrun/PierreDiffsSwift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
