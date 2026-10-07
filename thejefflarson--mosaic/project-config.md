---
trigger: always_on
description: macOS desktop app: an infinite 2D spatial canvas where real PTY-backed terminal windows can be freely panned, zoomed, dragged, and resized. Built in Swift + AppKit + SwiftTerm. No web/JS runtime.
---

# CanvasTerm — Claude Code Context

## What this is

macOS desktop app: an infinite 2D spatial canvas where real PTY-backed terminal windows can be freely panned, zoomed, dragged, and resized. Built in Swift + AppKit + SwiftTerm. No web/JS runtime.

## Build

```bash
# Generate Xcode project (required after editing project.yml)
xcodegen generate

# Build from command line
xcodebuild build -scheme Mosaic -destination 'platform=macOS' -IDEPackageSupportDisableManifestSandbox=1

# Run tests
xcodebuild test -scheme MosaicTests -destination 'platform=macOS' -IDEPackageSupportDisableManifestSandbox=1

# Open in Xcode, then Cmd+B or Cmd+R
open Mosaic.xcodeproj
```

> **Note:** `-IDEPackageSupportDisableManifestSandbox=1` is required when running xcodebuild from a shell where `sandbox-exec` is blocked at the kernel level (e.g. inside Claude Code / ziplock).
>
> **ziplock one-time setup:** If running Claude Code under [ziplock](https://github.com/thejefflarson/ziplock), run this once on the host to suppress Xcode's own Package.swift manifest sandbox:
> ```bash
> defaults write com.apple.dt.Xcode IDEPackageSupportDisableManifestSandbox -bool YES
> ```
> ziplock sets `XBS_DISABLE_SANDBOXED_BUILDS=1` and `SWIFTPM_SANDBOX=0` automatically. The release script already passes `-IDEPackageSupportDisableManifestSandbox=1` to both xcodebuild invocations.

## Project structure

```
CanvasTerm/
├── App/                    # Entry point, AppDelegate, menu bar, ScriptingCommands
├── Canvas/                 # CanvasView (pan/zoom), CanvasViewController, CanvasGeometry
├── Terminal/               # TerminalController, TerminalWindowView, TitleBarView,
│                           #   ResizeHandleView, TerminalManager, TerminalSettings
├── Annotations/            # AnnotationController, AnnotationViews, CanvasTool,
│                           #   ToolPaletteView
├── Minimap/                # MinimapView overlay
├── Theming/                # Theme, ThemeEditorViewController
├── Persistence/            # WorkspaceSnapshot (Codable), WorkspaceStore, WorkspaceState
└── Utilities/              # CGRect/CGPoint extensions, SnapEngine, CanvasCursorManager
```

## Key architecture decisions

**Pan/zoom:** `CanvasView` holds a `FlippedView` (world space). Pan/zoom is applied as a `CGAffineTransform` on `worldView.layer`. All terminal windows are subviews positioned in world coordinates. No per-frame CPU work — CoreAnimation composites via Metal.

**Zoom-to-cursor math** is in `CanvasGeometry.swift` (`Viewport.zoomAround`). The invariant: world point under the cursor stays fixed.

**Viewport culling:** `CanvasView.updateCulling()` hides (`isHidden = true`) any terminal whose world-space frame doesn't intersect the visible rect. PTY process stays alive; only rendering is skipped.

**SwiftTerm integration:** `LocalProcessTerminalView` handles PTY spawn, I/O, and SIGWINCH. Do not override `terminalDelegate` externally — `LocalProcessTerminalView` sets it internally to pipe keystrokes to the PTY. Broadcast interception uses `InterceptingTerminalView` (subclass), which calls `super.send()` first.

**Concurrency:** Swift 6, `targeted` strict concurrency. All AppKit/UI work is `@MainActor`. `LocalProcessTerminalViewDelegate` methods are `nonisolated` and dispatch to main via `Task { @MainActor in }`. `WorkspaceStore` is `Sendable`; snapshots are value types passed to a serial `DispatchQueue`.

**Drag in world space:** Screen-space delta from `mouseDragged` must be divided by `currentZoom` before applying to world-space frame origin. Forgetting this makes windows "run away" at non-100% zoom.

**FlippedView:** `worldView` is a `FlippedView` (isFlipped = true) so Y increases downward, matching screen coordinate conventions for pan math.

**Snap engine:** `SnapEngine.swift` is a pure function (`snapRect`) that tests all 9 edge combinations (minX, midX, maxX × minY, midY, maxY) against reference rects and returns the best snap plus world-space guide line positions. `CanvasViewController.snapPosition()` calls it, converting guide positions to screen space for `SnapGuideOverlay`. During group drag, the bounding box of all selected items is snapped (not the dragged item's frame alone), and the resulting correction delta is applied uniformly to all peers.

**Controller pattern:** `TerminalController` and `AnnotationController` own their respective element sets and are created by `CanvasViewController`. Dependencies (snap, undo, theme) are injected as closures at init time so controllers never hold a back-reference to the VC. Mutations fire an `onChange` closure; the VC wires this to minimap refresh, selection ring update, and the 5-second save debounce.

**Undo model:** Undo actions are registered directly on `NSUndoManager` (via the responder chain) without grouping — each mutation is a single undoable step. Group-drag peer undos are registered in the same `mouseUp` event as the dragged item, so NSUndoManager's event-based coalescing bundles them into one Cmd+Z action automatically.

## Dependency


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thejefflarson/mosaic](https://github.com/thejefflarson/mosaic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
