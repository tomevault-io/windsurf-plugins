---
trigger: always_on
description: This file provides guidance for Claude Code when working on this project.
---

# CLAUDE.md

This file provides guidance for Claude Code when working on this project.

## Project Overview

Pesto Clipboard is a macOS menu bar clipboard manager built with SwiftUI. It stores clipboard history (text, images, files) and allows quick access via global hotkey.

## Build Commands

```bash
# Build the project
xcodebuild -project PestoClipboard/PestoClipboard.xcodeproj -scheme PestoClipboard -configuration Debug build

# Run tests
xcodebuild -project PestoClipboard/PestoClipboard.xcodeproj -scheme PestoClipboard test

# Build for release
xcodebuild -project PestoClipboard/PestoClipboard.xcodeproj -scheme PestoClipboard -configuration Release build
```

## Makefile Commands

The project includes a Makefile with convenient shortcuts:

```bash
make build          # Build release version
make build-debug    # Build debug version
make test           # Run tests
make install        # Build and copy to /Applications
make dmg            # Create distributable DMG
make clean          # Remove build artifacts
make bump-version V=0.3.4  # Bump version number
make locales-export # Export translations to markdown
make locales-import # Import translations from markdown
make locales-status # Show translation status
```

## Localization

Translations are managed through markdown files as the source of truth. The workflow uses `scripts/xcstrings.py` to sync between Xcode's `Localizable.xcstrings` and human-editable markdown files in `locales/`.

### File Structure

```
scripts/xcstrings.py              # CLI tool for managing translations
locales/
  de.md                           # German translations
  fr.md                           # French translations
  id.md                           # Indonesian translations
  ...                             # One file per language
PestoClipboard/PestoClipboard/
  Localizable.xcstrings           # Xcode's string catalog (generated)
```

### Workflow

```
xcstrings → export → locales/*.md → edit/review/PR → import → xcstrings
```

### Editing Translations

1. Export current state: `./scripts/xcstrings.py export`
2. Edit the markdown file (e.g., `locales/de.md`)
3. Fill in translations in the table's language column
4. Preview changes: `./scripts/xcstrings.py import --dry-run`
5. Apply changes: `./scripts/xcstrings.py import`
6. Verify build: `make build-debug`

### Adding a New Language

1. Add language code to `LANGUAGES` dict in `scripts/xcstrings.py`
2. Export: `./scripts/xcstrings.py export --lang <code>`
3. Fill in translations in `locales/<code>.md`
4. Import: `./scripts/xcstrings.py import --lang <code>`

### Markdown Format

Each language file uses a table format:

```markdown
| Status | English | German |
|--------|---------|--------|
| ✓ | About | Über |
| ✗ | Settings |  |
| ⚠ | Help | Hilfe |
```

- `✓` = translated
- `✗` = missing (empty target column)
- `⚠` = stale (source changed, needs review)

### CLI Commands

```bash
./scripts/xcstrings.py export                # Export all languages
./scripts/xcstrings.py export --lang de      # Export single language
./scripts/xcstrings.py import                # Import all changes
./scripts/xcstrings.py import --lang de      # Import single language
./scripts/xcstrings.py import --dry-run      # Preview changes
./scripts/xcstrings.py status                # Show translation summary
./scripts/xcstrings.py validate              # Check xcstrings integrity
```

### Supported Languages

ca (Catalan), da (Danish), de (German), en-GB (English UK), es (Spanish), fr (French), hi (Hindi), id (Indonesian), it (Italian), ja (Japanese), ko (Korean), nl (Dutch), ru (Russian), sv (Swedish), zh-Hans (Chinese Simplified)

## Architecture

### Key Components

- **App/AppDelegate.swift** - Sets up global hotkey and initializes StatusBarController
- **App/AppEventBus.swift** - Type-safe Combine-based event bus for inter-component communication
- **Services/ClipboardMonitor.swift** - Polls clipboard for changes every 0.5s
- **Services/ClipboardHistoryManager.swift** - Core Data CRUD operations for clipboard items
- **Services/SettingsManager.swift** - UserDefaults-backed settings (singleton)
- **Views/StatusBar/StatusBarController.swift** - Menu bar icon orchestration
- **Views/StatusBar/FloatingPanel.swift** - NSPanel subclass for the history popup
- **Views/StatusBar/EventMonitorManager.swift** - Global/local event monitoring
- **Views/StatusBar/PreferencesWindowController.swift** - Preferences window lifecycle
- **Views/HistoryPopover/HistoryView.swift** - Main UI for clipboard history list
- **Views/HistoryPopover/HistoryViewModel.swift** - State and business logic for HistoryView
- **Views/HistoryPopover/HistoryKeyboardHandlers.swift** - Keyboard shortcut handling

### Data Flow

1. ClipboardMonitor detects clipboard changes
2. ClipboardHistoryManager saves items to Core Data
3. HistoryView displays items via @ObservedObject binding
4. FloatingPanel shows/hides via global hotkey or menu bar click

### Focus System

The app uses a custom focus system to handle keyboard shortcuts:
- `KeyAcceptingHostingView` (in FloatingPanel.swift) - Custom NSHostingView subclass that accepts first responder
- `FocusField` enum in HistoryView - `.list` (default) or `.search`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matthewpick/pesto-clipboard](https://github.com/matthewpick/pesto-clipboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
