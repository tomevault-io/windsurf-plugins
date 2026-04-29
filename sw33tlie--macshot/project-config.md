---
trigger: always_on
description: Native macOS screenshot & annotation tool inspired by Flameshot. Built with Swift + AppKit. No Qt, no Electron.
---

# macshot

Native macOS screenshot & annotation tool inspired by Flameshot. Built with Swift + AppKit. No Qt, no Electron.

## Project Setup

- **Language:** Swift 5.0
- **UI:** AppKit (all windows created in code, storyboard is minimal — just app entry + main menu)
- **Min Target:** macOS 12.3+ (Monterey)
- **Bundle ID:** com.sw33tlie.macshot.macshot
- **Sandbox:** Enabled (entitlements: network.client, files.user-selected.read-write, files.bookmarks.app-scope)
- **LSUIElement:** YES (menu bar only app, no dock icon — switches to `.regular` when editor windows are open)
- **Permissions:** Screen Recording (Info.plist has Privacy - Screen Capture Usage Description)
- **Xcode:** File system synchronized groups — just create .swift files in `macshot/` and Xcode picks them up automatically

## Architecture

Menu bar agent app. No main window. Global hotkey (Cmd+Shift+X) or menu bar click triggers screen capture → fullscreen overlay → selection → annotation → output.

### File Structure

```
macshot/
├── main.swift                          # App entry point
├── AppDelegate.swift                   # App lifecycle, status bar, hotkey, capture orchestration
│
├── Model/
│   └── Annotation.swift                # Data model + drawing for all annotation types
│
├── Capture/
│   ├── ScreenCaptureManager.swift      # Multi-screen capture via ScreenCaptureKit (async/await)
│   ├── RecordingEngine.swift           # Screen recording (MP4 via AVAssetWriter, GIF via GIFEncoder)
│   ├── ScrollCaptureController.swift   # Scroll capture with SAD-based stitching
│   └── GIFEncoder.swift               # Animated GIF from video frames
│
├── Services/
│   ├── ImageEncoder.swift              # PNG/JPEG/HEIC/WebP encoding, clipboard copy, resolution scaling
│   ├── BeautifyRenderer.swift          # Gradient frame / background beautification (linear + mesh gradients)
│   ├── AutoRedactor.swift              # PII regex detection + Vision OCR → redaction annotations
│   ├── BarcodeDetector.swift           # Async Vision barcode/QR scanning, badge drawing, hit-testing
│   ├── TranslationOverlay.swift        # OCR → translate → overlay annotations
│   ├── TranslationService.swift        # Google Translate API wrapper
│   ├── VisionOCR.swift                 # Vision text recognition request factory
│   ├── HotkeyManager.swift            # Global keyboard shortcut (Carbon RegisterEventHotKey)
│   ├── ScreenshotHistory.swift         # Local history in ~/Library/Application Support/
│   └── SaveDirectoryAccess.swift       # Security-scoped bookmark for save directory
│
├── Upload/
│   ├── ImgbbUploader.swift             # imgbb image upload
│   ├── GoogleDriveUploader.swift       # Google Drive OAuth2 upload
│   └── S3Uploader.swift               # S3-compatible upload
│
├── UI/
│   ├── Overlay/
│   │   ├── OverlayView.swift           # Base canvas: selection, drawing, annotation rendering, input routing
│   │   ├── OverlayView+Popovers.swift  # Popover factories + auto-redact/translate action helpers
│   │   ├── OverlayView+Recording.swift # Recording HUD, mouse highlight monitor
│   │   ├── OverlayView+ScrollCaptureHUD.swift  # Scroll capture progress bar + stop button
│   │   ├── OverlayView+WindowSnapping.swift    # Window detection + snap highlight drawing
│   │   ├── OverlayWindowController.swift       # One per screen: fullscreen borderless overlay window
│   │   └── ColorWheelRenderer.swift    # Radial color wheel for right-click quick color pick
│   │
│   ├── Editor/
│   │   ├── EditorView.swift            # OverlayView subclass: NSScrollView mode, no selection chrome
│   │   ├── DetachedEditorWindowController.swift  # Standalone editor window (resizable, titled)
│   │   ├── EditorTopBarView.swift      # NSView with crop, flip, zoom buttons
│   │   ├── CenteringClipView.swift     # NSClipView subclass that centers document when smaller than clip
│   │   └── VideoEditorWindowController.swift  # Standalone video editor (trim, export, upload)
│   │
│   ├── Toolbar/
│   │   ├── ToolbarDefinitions.swift    # ToolbarButtonAction enum, ToolbarButton struct, ToolbarLayout constants
│   │   ├── ToolbarButtonView.swift     # NSView for a single toolbar button (hover, press, selection states)
│   │   ├── ToolbarStripView.swift      # NSView container for horizontal/vertical button rows
│   │   └── ToolOptionsRowView.swift    # NSView-based tool options bar (sliders, segments, text formatting)
│   │
│   ├── Tools/
│   │   ├── AnnotationToolHandler.swift # AnnotationToolHandler + AnnotationCanvas protocols, shared helpers
│   │   ├── PencilToolHandler.swift     # Freeform draw with Chaikin smoothing
│   │   ├── MarkerToolHandler.swift     # Highlighter (semi-transparent wide stroke)
│   │   ├── LineToolHandler.swift       # Straight line with 45° snap
│   │   ├── ArrowToolHandler.swift      # Arrow with styles (single, thick, double, open, tail)
│   │   ├── RectangleToolHandler.swift  # Rectangle with corner radius, fill style, line style
│   │   ├── FilledRectangleToolHandler.swift  # Opaque filled rectangle (redact)
│   │   ├── EllipseToolHandler.swift    # Ellipse with fill style
│   │   ├── PixelateToolHandler.swift   # Pixelate region

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sw33tLie/macshot](https://github.com/sw33tLie/macshot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
