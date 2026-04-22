---
trigger: always_on
description: A Python framework for building native macOS menu bar applications using SwiftUI.
---

# Nib

A Python framework for building native macOS menu bar applications using SwiftUI.

## Architecture

Two processes, one socket. Python owns the logic, Swift owns the screen. They communicate over a Unix socket using MessagePack. In dev mode Python spawns Swift; in a built `.app` Swift spawns Python.

- `render` — full view tree, sent once at startup
- `patch` — only what changed, sent on every update
- `event` — user interaction (tap, change, hover), sent from Swift to Python

See `architecture.md` for a detailed overview.

## Project Structure

```
nib/
├── sdk/python/nib/           # Python SDK
│   ├── core/
│   │   ├── app.py            # App class, SFSymbol, MenuItem, run()
│   │   ├── connection.py     # Unix socket client, MessagePack
│   │   ├── diff.py           # View tree diffing, patch generation
│   │   ├── settings.py       # Settings with UserDefaults persistence
│   │   ├── user_defaults.py  # Low-level UserDefaults access
│   │   ├── state.py          # Reactive State and Binding
│   │   ├── logging.py        # Logger with progress bar support
│   │   └── file_picker.py   # Open/save file dialogs
│   ├── views/
│   │   ├── controls/         # Text, Button, TextField, Toggle, Slider, Picker,
│   │   │                     # DatePicker, ColorPicker, Image, Video, Label, Link,
│   │   │                     # ProgressView, Gauge, Markdown, Table, SecureField,
│   │   │                     # TextEditor, Map, WebView, ShareLink, CameraPreview
│   │   ├── layout/           # VStack, HStack, ZStack, ScrollView, List, Section,
│   │   │                     # Form, Spacer, Divider, Group, NavigationStack,
│   │   │                     # NavigationLink, DisclosureGroup, Grid, LazyVGrid, LazyHGrid
│   │   ├── shapes/           # Rectangle, Circle, Ellipse, Capsule
│   │   ├── charts/           # Chart, LineMark, BarMark, AreaMark, PointMark,
│   │   │                     # RuleMark, RectMark, SectorMark
│   │   ├── effects/          # VisualEffectBlur
│   │   ├── canvas.py         # Canvas (Core Graphics drawing)
│   │   └── settings_page.py  # SettingsPage, SettingsTab
│   ├── cli/
│   │   ├── build.py          # nib build — .app bundling, compilation, signing
│   │   ├── run.py            # nib run — dev mode with hot reload
│   │   ├── create.py         # nib create — project scaffolding
│   │   └── obfuscate.py      # Bytecode obfuscation
│   ├── services/             # Battery, Connectivity, Screen, Keychain, Camera, LaunchAtLogin
│   ├── notifications/        # Notification, NotificationManager, scheduling, actions
│   ├── draw/                 # Canvas drawing primitives
│   ├── modifiers/            # Layout, appearance, typography, effects
│   └── types.py              # Color, Font, Animation, TextStyle, enums
│
├── package/Nib/              # Swift runtime
│   ├── App/
│   │   ├── AppDelegate.swift       # Entry point, message routing, bundled mode
│   │   ├── Handlers/               # ClipboardHandler, FileDialogHandler, UserDefaultsHandler
│   │   ├── Managers/               # HotkeyManager
│   │   └── Services/               # Battery, Camera, Connectivity, Keychain,
│   │                               # LaunchAtLogin, Notification, Screen
│   ├── Core/
│   │   ├── Plugins/                # PluginLoader, PluginProtocol
│   │   └── Registries/             # ViewBuilderRegistry, ModifierRegistry
│   ├── Network/
│   │   └── SocketServer.swift      # Unix socket server, MessagePack parsing
│   ├── Protocol/
│   │   ├── NibMessage.swift        # Message types
│   │   ├── ViewNode.swift          # View tree structure
│   │   ├── ViewModifier.swift      # Modifier types
│   │   ├── ViewProps.swift         # View property definitions
│   │   ├── DrawCommand.swift       # Canvas drawing commands
│   │   └── Types/                  # ChartTypes, LayoutTypes, MapTypes, etc.
│   └── UI/
│       ├── StatusBarController.swift       # Menu bar icon, popover, context menu
│       ├── SettingsWindowController.swift  # Settings window
│       ├── ViewStore.swift                 # Observable state for SwiftUI
│       ├── FontManager.swift               # Font loading
│       └── Rendering/
│           ├── DynamicView.swift     # Main SwiftUI renderer
│           ├── Builders/             # 18 builder files (per view type)
│           └── Modifiers/            # Appearance, layout, animation, transition, font
│
└── examples/
```

## Building

### Swift Runtime

```bash
cd package && swift build -c release
```

Binary: `package/.build/release/nib-runtime`

### Using Makefile

```bash
make build-runtime  # Build and copy runtime to SDK
make install        # Build and install in dev mode
```

## CLI

```bash
nib create myapp        # Scaffold a new project
nib run main.py         # Dev mode with hot reload (-r for recursive watch)
nib build main.py       # Build standalone .app bundle
nib build --native      # Compile to native .so via Cython
nib build --obfuscate   # Strip debug info from .pyc
nib build --no-compile  # Keep .py source files
```

Build config can go in `pyproject.toml` under `[tool.nib]` and `[tool.nib.build]`.

## API

### Function-based (recommended)

```python
import nib

def main(app: nib.App):
    app.title = "My App"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Bbalduzz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
