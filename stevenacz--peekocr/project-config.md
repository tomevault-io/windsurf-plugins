---
trigger: always_on
description: PeekOCR is a native macOS Menu Bar application for OCR text capture, QR code detection, screenshots, **GIF clip recording**, and **annotation editing**. Built with Swift 5.9, SwiftUI, and AppKit.
---

# AGENTS.md - PeekOCR

## Project Overview

PeekOCR is a native macOS Menu Bar application for OCR text capture, QR code detection, screenshots, **GIF clip recording**, and **annotation editing**. Built with Swift 5.9, SwiftUI, and AppKit.

## Documentation Index

| Document | Description |
|----------|-------------|
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | Layer diagram, data flow, patterns |
| [docs/COMPONENTS.md](docs/COMPONENTS.md) | Reusable UI components catalog |
| [docs/GIF_CLIP.md](docs/GIF_CLIP.md) | GIF clip capture flow + key modules |
| [docs/MODELS.md](docs/MODELS.md) | Data models and state managers |
| [docs/SERVICES.md](docs/SERVICES.md) | Service classes documentation |
| [docs/VIEWS.md](docs/VIEWS.md) | View hierarchy and modules |

## Dev Environment

### Requirements

- Xcode 15+
- macOS 13.0+ (Ventura)
- Swift 5.9

### Build Commands

```bash
# Debug build
xcodebuild -project PeekOCR.xcodeproj -scheme PeekOCR -configuration Debug build

# Release build
xcodebuild -project PeekOCR.xcodeproj -scheme PeekOCR -configuration Release build

# Clean build folder
xcodebuild clean -project PeekOCR.xcodeproj -scheme PeekOCR

# Open in Xcode
open PeekOCR.xcodeproj
```

## Project Structure (Final)

```
PeekOCR/
├── Models/
│   ├── Annotations/                  # Annotation data types
│   │   ├── Annotation.swift          # Single annotation model
│   │   ├── AnnotationTool.swift      # Tool enum (arrow, text, etc.)
│   │   └── ResizeHandle.swift        # Resize handle positions
│   │
│   ├── State/                        # State managers
│   │   ├── UndoRedoManager.swift     # Generic undo/redo
│   │   ├── SelectionManager.swift    # Selection state
│   │   ├── TextInputController.swift # Text input lifecycle
│   │   ├── AnnotationUndoManager.swift   # Annotation-specific undo/redo (NEW)
│   │   ├── AnnotationDragManager.swift   # Drag and resize state (NEW)
│   │   ├── AnnotationTextManager.swift   # Text input state (NEW)
│   │   └── GifClipEditorState.swift      # GIF trim + playback state
│   │
│   ├── AnnotationState.swift         # Main editor state (refactored, uses composition)
│   ├── AppState.swift                # Global app state
│   ├── AppSettings.swift             # User preferences
│   ├── AnnotationSettings.swift      # Annotation defaults
│   ├── ScreenshotSettings.swift      # Screenshot options
│   ├── CaptureItem.swift             # History item
│   ├── GifExportOptions.swift         # GIF export presets/options
│   └── SaveLocation.swift            # Save location enum
│
├── Services/
│   ├── Annotation/
│   │   ├── AnnotationGeometry.swift       # Geometry calculations
│   │   └── AnnotationWindowFactory.swift  # Window creation
│   │
│   ├── Screenshot/
│   │   ├── ImageScalingService.swift      # High-quality scaling
│   │   └── ImageEncodingService.swift     # Format conversion
│   │
│   ├── HotKey/
│   │   └── HotKeyDefinition.swift         # Hotkey config struct
│   │
│   ├── AnnotationWindowController.swift   # Window lifecycle
│   ├── CaptureCoordinator.swift           # Capture orchestration
│   ├── GifClipWindowController.swift      # GIF editor window lifecycle
│   ├── GifClipWindowFactory.swift         # GIF editor window creation
│   ├── GifExportService.swift             # Video -> GIF export
│   ├── GifRecordingController.swift       # Selection + recording orchestration
│   ├── GifRecordingHudWindowController.swift     # Countdown + stop HUD
│   ├── GifRecordingOverlayWindowController.swift # Full-screen selection overlay
│   ├── ScreenshotService.swift            # Screenshot processing
│   ├── NativeScreenCaptureService.swift   # Native capture
│   ├── NativeScreenRecordingService.swift # Native video capture support checks
│   ├── HotKeyManager.swift                # Global shortcuts
│   └── OCRService.swift                   # Text recognition
│
├── Views/
│   ├── Annotation/
│   │   ├── Canvas/
│   │   │   ├── AnnotationCanvasView.swift    # Drawing surface (~182 lines)
│   │   │   ├── AnnotationRenderer.swift      # Draw dispatcher
│   │   │   ├── SelectionHandlesView.swift    # Resize handles
│   │   │   └── TextInputOverlay.swift        # Text input field
│   │   │
│   │   ├── Editor/
│   │   │   ├── AnnotationEditorView.swift    # Main editor (~125 lines)
│   │   │   ├── KeyboardEventHandler.swift    # Keyboard shortcuts
│   │   │   └── CGContextAnnotationRenderer.swift  # Export rendering
│   │   │
│   │   ├── Toolbar/
│   │   │   ├── AnnotationToolbar.swift       # Main toolbar
│   │   │   ├── ToolButton.swift              # Tool selection
│   │   │   ├── ActionIconButton.swift        # Action buttons
│   │   │   └── ShortcutKeyBadge.swift        # Shortcut display
│   │   │
│   │   ├── ColorPaletteView.swift
│   │   └── StrokeWidthPicker.swift
│   │
│   ├── MenuBar/
│   │   ├── MenuBarPopoverView.swift          # Main popover (~179 lines)
│   │   ├── MenuBarActionButton.swift         # Quick action button
│   │   ├── HistoryItemRow.swift              # History item row
│   │   └── EmptyStateView.swift              # Empty state placeholder
│   │
│   ├── Gif/
│   │   ├── GifClipEditorView.swift           # Post-record editor window
│   │   ├── GifClipKeyboardHandler.swift      # Frame-by-frame shortcuts
│   │   ├── GifClipPlaybackControlsView.swift # Play/pause + stepping UI
│   │   ├── GifClipSidebarView.swift          # Export options + estimates
│   │   ├── GifClipTimelineReadoutView.swift  # In/out readout
│   │   ├── GifClipTimelineView.swift         # Timeline + trim range
│   │   ├── GifClipVideoPreviewView.swift     # Video preview container
│   │   ├── GifExportLoadingOverlay.swift     # Export loading UI
│   │   └── Overlay/
│   │       ├── GifRecordingHudView.swift     # HUD SwiftUI view
│   │       └── GifRecordingOverlayView.swift # Selection overlay view
│   │
│   ├── Settings/
│   │   ├── Sections/                         # Modular settings sections
│   │   │   ├── HotkeyDisplaySection.swift    # Hotkey display
│   │   │   ├── SaveOptionsSection.swift      # Save toggles
│   │   │   ├── SaveLocationSection.swift     # Location picker
│   │   │   ├── ImageFormatSection.swift      # Format picker
│   │   │   ├── ImageScaleSection.swift       # Scale slider
│   │   │   └── AnnotationDefaultsSection.swift  # Default settings
│   │   │
│   │   ├── SettingsView.swift
│   │   ├── GeneralSettingsTab.swift
│   │   ├── ShortcutsSettingsTab.swift
│   │   ├── ScreenshotSettingsTab.swift       # Uses sections (~36 lines)
│   │   ├── HistorySettingsTab.swift
│   │   └── AboutTab.swift
│   │
│   ├── Components/                           # Reusable components
│   │   ├── NonInteractiveVideoPlayer.swift   # AVPlayerView wrapper (no controls)
│   │   ├── RangeSlider.swift                 # Dual-handle range slider
│   │   ├── SectionHeader.swift
│   │   ├── SectionDivider.swift
│   │   ├── SettingSliderRow.swift
│   │   ├── ShortcutRecorderRow.swift
│   │   └── PermissionStatusRow.swift
│   │
│   └── Helpers/
│       ├── HitTestEngine.swift               # Collision detection
│       └── AnnotationTransformer.swift       # Move/resize operations
│
├── Managers/
│   └── HistoryManager.swift                  # Capture history (singleton)
│
├── Utils/
│   └── AppLogger.swift                       # Centralized logging (OSLog)
│
├── Extensions/
│   ├── CaptureType+Display.swift
│   └── HotKeyDisplay.swift
│
├── AppDelegate.swift
├── PeekOCRApp.swift
└── Constants.swift

```

## Key Architecture

### Patterns Used

- **MVVM**: Views observe ObservableObject state
- **Coordinator**: CaptureCoordinator orchestrates capture flow
- **Singleton**: Shared services (`CaptureCoordinator.shared`)
- **Factory**: `AnnotationWindowFactory` creates configured windows
- **Static Helpers**: Pure functions in enums (`AnnotationGeometry`, `HitTestEngine`, `ImageScalingService`)
- **Composition**: AnnotationState delegates to specialized managers (UndoManager, DragManager, TextManager)
- **Centralized Logging**: AppLogger with OSLog for structured debugging

### Component Guidelines

- Each file starts with 1-line English description comment
- Max ~80-150 lines per component (exceptions documented)
- Single responsibility per file
- Reusable components in `Views/Components/`
- Settings sections in `Views/Settings/Sections/`
- Include `#Preview` for visual testing

### Naming Conventions

| Type | Pattern | Example |
|------|---------|---------|
| Views | `*View.swift`, `*Section.swift` | `AnnotationCanvasView.swift` |
| Services | `*Service.swift`, `*Manager.swift` | `ScreenshotService.swift` |
| Factories | `*Factory.swift` | `AnnotationWindowFactory.swift` |
| Helpers | `*Engine.swift`, `*Transformer.swift` | `HitTestEngine.swift` |
| Components | Descriptive noun | `ToolButton.swift` |

## Capture Modes

| Mode | Hotkey | Action |
|------|--------|--------|
| OCR | `⇧ Space` | Extract text, copy to clipboard |
| Screenshot | `⌘⇧4` | Save image to file |
| Annotated | `⌘⇧5` | Open annotation editor, then save |
| GIF Clip | `⌘⇧6` | Select region, record up to 10s, export as GIF |

## Common Tasks

### Adding a New Capture Mode
1. Add case to `CaptureMode` in `Services/CaptureCoordinator.swift`
2. Add a quick action button in `Views/MenuBar/MenuBarPopoverView.swift`
3. Register the default hotkey in `Models/AppSettings.swift` and `Services/HotKeyManager.swift`
4. Add/label the shortcut in `Views/Settings/ShortcutsSettingsTab.swift`
5. Update docs and manual QA checklist

### Adding a New Annotation Tool
1. Add case to `AnnotationTool` enum
2. Add icon, display name, shortcut key
3. Add drawing in `AnnotationRenderer`
4. Add export in `CGContextAnnotationRenderer`
5. Update `KeyboardEventHandler` if needed

### Adding a New Setting
1. Add property to settings model
2. Create section in `Views/Settings/Sections/` if needed
3. Add to settings tab
4. Use `@AppStorage` for persistence

### Adding a New Component
1. Create in `Views/Components/` or `Views/Settings/Sections/`
2. Add English description comment
3. Make reusable with parameters
4. Document in `docs/COMPONENTS.md`

### Adding a New Service
1. Create in appropriate subfolder (`Services/Screenshot/`, `Services/HotKey/`, etc.)
2. Add English description comment
3. Use static methods for pure operations
4. Document in `docs/SERVICES.md`

## Frameworks

| Framework | Purpose |
|-----------|---------|
| SwiftUI | UI components |
| AppKit | Menu bar, windows |
| Vision | OCR, QR detection |
| Carbon | Global hotkeys |
| CoreGraphics | Image processing |
| CoreText | Text rendering |
| AVFoundation / AVKit | Video preview + frame extraction |
| ImageIO | GIF encoding |
| UniformTypeIdentifiers | GIF UTType identifiers |

## Permissions Required

- **Screen Recording**: For capture
- **Accessibility**: For global hotkeys

## Testing Checklist

- [ ] Menu bar icon appears
- [ ] Hotkeys trigger capture
- [ ] Annotation editor opens for `⌘⇧5`
- [ ] GIF clip capture opens for `⌘⇧6` (select → record → editor)
- [ ] Drawing tools work (arrow, text, freehand, rectangle, oval)
- [ ] Undo/redo works
- [ ] Selection and resize works
- [ ] Save exports image correctly
- [ ] GIF export saves correctly and appears in history
- [ ] Settings persist

## Git Workflow

```bash
# Before committing
xcodebuild -scheme PeekOCR build

# Commit format
<type>: <description>
# Types: feat, fix, refactor, style, docs, chore
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/StevenACZ)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/StevenACZ)
<!-- tomevault:4.0:windsurf_rules:2026-04-08 -->
