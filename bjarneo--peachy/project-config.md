---
trigger: always_on
description: Terminal UI (TUI) application for extracting color palettes from images. Built with Go using the Charmbracelet stack.
---

# Peachy

Terminal UI (TUI) application for extracting color palettes from images. Built with Go using the Charmbracelet stack.

## Tech Stack

- **Bubbletea** - TUI framework (Elm architecture)
- **Lipgloss** - Styling and layout
- **Bubbles** - Reusable components (key bindings)
- **Cobra** - CLI framework

## Project Structure

```
internal/
├── shared/                 # Shared utilities and constants
│   ├── constants.go        # Color indices, UI dimensions, cache settings
│   ├── images.go           # IsValidImage() - unified image validation
│   └── status.go           # StatusType enum (Info/Success/Error/Warning)
├── ui/
│   ├── model.go            # Main app model, Update/View, state machine
│   ├── styles.go           # ANSI color definitions and lipgloss styles
│   ├── keys.go             # KeyMap struct with key.Binding definitions
│   └── components/
│       ├── colorlist.go    # 16-color palette list (Normal/Bright sections)
│       ├── coloreditor.go  # HSL sliders + hex input modal
│       ├── filepicker.go   # File browser with FileFilter interface
│       ├── preview.go      # Image + color swatch preview
│       ├── imagepreview.go # Unicode block image rendering
│       └── help.go         # Help overlay
├── color/
│   ├── extractor.go        # Color extraction from images (ImageMagick)
│   ├── palette.go          # 16-color palette management
│   ├── modes.go            # Extraction modes + ParseMode()
│   ├── convert.go          # Color space conversions (RGB/HSL/Hex)
│   ├── generators.go       # Palette generation algorithms
│   └── analysis.go         # Color analysis utilities
├── config/
│   ├── paths.go            # XDG paths, theme directories
│   ├── toml.go             # Theme file I/O
│   ├── templates.go        # System theme generation
│   └── cli.go              # CLI output helpers (PrintColoredPalette, etc.)
├── cache/
│   └── thumbnails.go       # LRU thumbnail cache with error collection
└── terminal/
    └── blocks.go           # Unicode block rendering with cache
```

## View States

The app uses a state machine with these views:
- `ViewMain` - Split pane: color list + preview
- `ViewEditor` - Color editing modal (HSL/hex)
- `ViewFilePicker` - File browser
- `ViewHelp` - Keyboard shortcuts overlay
- `ViewSaveTheme` - Theme name input
- `ViewThemeBrowser` - Load/apply saved themes

## Key Patterns

- **Status handling**: Uses `shared.Status` type with `StatusType` enum instead of string matching
- **Key bindings**: Uses `key.Matches(msg, m.keys.*)` with KeyMap struct from `ui/keys.go`
- **Image validation**: Centralized in `shared.IsValidImage()` - single source of truth
- **Constants**: Color indices (0-15), cache settings in `shared/constants.go`
- **FileFilter interface**: Extensible filtering in filepicker (`ImageFilter`, `SearchFilter`)
- **Cache**: LRU eviction (max 500), error collection for concurrent ops
- All UI uses ANSI colors 0-15 for theming consistency
- Vim-style navigation (j/k/h/l/g/G)
- Components implement `Init()`, `Update()`, `View()` pattern
- Live preview updates during color editing

## Build & Run

```bash
go build
./peachy
```

## Testing

```bash
go test ./...
```

Tests cover: color conversion, palette management, modes, status, image validation.

## Claude Code Instructions

- **Never run `git add` or `git commit`** - wait for explicit user request
- Always run tests before suggesting commits
- Follow existing patterns in the codebase
- Write docs in devrel style

## Key Bindings (Main View)

- `j/k` - Navigate colors
- `Enter` - Edit selected color
- `o` - Open image file picker
- `e` - Extract colors from image
- `m/M` - Cycle extraction mode
- `t` - Toggle light/dark mode
- `s` - Save theme
- `l` - Load theme
- `a` - Apply theme to system
- `?` - Help
- `q` - Quit

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bjarneo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
