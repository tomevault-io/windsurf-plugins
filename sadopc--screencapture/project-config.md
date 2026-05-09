---
trigger: always_on
description: ScreenCapture is a fast, lightweight macOS menu bar application for capturing and annotating screenshots. It uses Apple's ScreenCaptureKit API and provides annotation tools similar to professional screenshot utilities.
---

# CLAUDE.md - ScreenCapture

## Project Overview

ScreenCapture is a fast, lightweight macOS menu bar application for capturing and annotating screenshots. It uses Apple's ScreenCaptureKit API and provides annotation tools similar to professional screenshot utilities.

**Tech Stack:** Swift 6.2, SwiftUI + AppKit, ScreenCaptureKit, CoreGraphics
**Minimum macOS:** 13.0 (Ventura)
**Build System:** Xcode 15.0+

## Build & Run

```bash
# Open in Xcode and run (Cmd+R)
open ScreenCapture.xcodeproj

# Command line build
xcodebuild -project ScreenCapture.xcodeproj -scheme ScreenCapture

# Archive for distribution
xcodebuild archive -project ScreenCapture.xcodeproj -scheme ScreenCapture
```

**Required Permission:** Screen Recording (System Settings → Privacy & Security → Screen Recording)

## Project Structure

```
ScreenCapture/
├── App/                                    # Application entry point
│   ├── ScreenCaptureApp.swift             # @main SwiftUI app with AppDelegate adaptor
│   └── AppDelegate.swift                  # Central coordinator: lifecycle, hotkeys, capture flow
│
├── Features/
│   ├── Capture/                           # Screenshot capture system
│   │   ├── CaptureManager.swift           # Actor: thread-safe ScreenCaptureKit operations
│   │   ├── ScreenDetector.swift           # Actor: display enumeration with 5s caching
│   │   ├── SelectionOverlayWindow.swift   # Region selection UI overlay on all displays
│   │   └── DisplaySelector.swift          # Multi-monitor display selection
│   │
│   ├── Preview/                           # Screenshot editing/annotation
│   │   ├── PreviewWindow.swift            # NSPanel window for screenshot editing
│   │   ├── PreviewViewModel.swift         # @Observable state: tools, annotations, undo
│   │   ├── PreviewContentView.swift       # SwiftUI content view with toolbar
│   │   └── AnnotationCanvas.swift         # Drawing surface for annotation rendering
│   │
│   ├── Annotations/                       # Drawing tools system
│   │   ├── AnnotationTool.swift           # Protocol: beginDrawing, continueDrawing, endDrawing
│   │   ├── RectangleTool.swift            # Rectangle outlines with configurable stroke
│   │   ├── FreehandTool.swift             # Freehand drawing paths
│   │   ├── ArrowTool.swift                # Directional arrows
│   │   └── TextTool.swift                 # Text annotations with font/color options
│   │
│   ├── MenuBar/                           # Status bar integration
│   │   └── MenuBarController.swift        # @MainActor: status item + dropdown menu
│   │
│   └── Settings/                          # Preferences UI
│       ├── SettingsView.swift             # SwiftUI settings form
│       ├── SettingsViewModel.swift        # @Observable settings state
│       └── SettingsWindowController.swift # NSWindow controller for preferences
│
├── Services/                              # Business logic layer
│   ├── ImageExporter.swift                # PNG/JPEG encoding, save to disk/clipboard
│   ├── HotkeyManager.swift                # Actor: global hotkey registration (Carbon APIs)
│   ├── ClipboardService.swift             # NSPasteboard operations
│   └── RecentCapturesStore.swift          # Recent captures history with thumbnails
│
├── Models/                                # Data types
│   ├── Screenshot.swift                   # Immutable: CGImage, metadata, annotations
│   ├── Annotation.swift                   # Enum: Rectangle, Freehand, Arrow, Text subtypes
│   ├── AppSettings.swift                  # @Observable singleton: UserDefaults preferences
│   ├── DisplayInfo.swift                  # Display metadata (resolution, scale, position)
│   ├── ExportFormat.swift                 # PNG/JPEG format enum with quality settings
│   ├── KeyboardShortcut.swift             # Hotkey representation (key + modifiers)
│   └── Styles.swift                       # StrokeStyle, TextStyle value types
│
├── Extensions/                            # Swift extensions
│   ├── CGImage+Extensions.swift           # CGImage helpers (cropping, scaling)
│   ├── NSImage+Extensions.swift           # NSImage ↔ CGImage conversions
│   └── View+Cursor.swift                  # SwiftUI cursor modifier
│
├── Errors/                                # Custom error types
│   └── ScreenCaptureError.swift           # Capture, permission, export errors
│
└── Resources/
    └── Assets.xcassets                    # App icon, colors, images
```

**File Count:** 34 Swift files across 11 directories

## Key Files

- **`App/AppDelegate.swift`** - Central coordinator for lifecycle, hotkeys, and capture flow
- **`Features/Capture/CaptureManager.swift`** - Actor for thread-safe ScreenCaptureKit operations
- **`Features/Capture/SelectionOverlayWindow.swift`** - Region selection UI
- **`Features/Preview/PreviewViewModel.swift`** - @Observable state management for annotations
- **`Features/Annotations/AnnotationTool.swift`** - Protocol for annotation tools
- **`Services/ImageExporter.swift`** - PNG/JPEG encoding and save operations
- **`Services/HotkeyManager.swift`** - Global hotkey registration (Carbon APIs)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sadopc/ScreenCapture](https://github.com/sadopc/ScreenCapture) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
