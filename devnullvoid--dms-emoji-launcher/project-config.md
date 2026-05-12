---
trigger: always_on
description: A DankMaterialShell (DMS) launcher plugin providing quick access to emojis, unicode characters, Latin extended characters, math symbols, and Nerd Font glyphs with instant clipboard copying.
---

# AGENTS.md - DMS Emoji & Unicode Launcher

## Project Overview
A DankMaterialShell (DMS) launcher plugin providing quick access to emojis, unicode characters, Latin extended characters, math symbols, and Nerd Font glyphs with instant clipboard copying.

**Language**: QML (Qt Modeling Language) with Python build scripts
**Type**: Launcher plugin for DankMaterialShell
**Default Trigger**: `:e`
**Version**: 1.5.2

## Recent Maintenance Notes (2026-03-01)
- Bumped plugin version to `1.5.2` for search-quality improvements.
- Moved legacy curated emoji/unicode seed arrays out of `EmojiLauncher.qml` into `defaultData.js` to keep QML maintainable while preserving robust keyword coverage.
- Updated search scoring to better prioritize exact emoji/unicode matches and demote Nerd Font symbols for generic queries.
- Updated catalog generation to filter filler stopwords (for example: `with`, `of`, `the`, `and`) from auto-generated keywords.
- Improved launcher search matching/ranking for multi-word queries:
  - tokenized matching across name/char/keywords (not just contiguous substring)
  - stronger ranking for language+letter intent (for example: `french e`, `latin tilde n`)
- Better relevance ordering for Latin Extended searches with single-letter tokens in multi-word queries.
- Existing behavior retained: default trigger is `:e`, configurable in settings.

## Previous Maintenance Notes (2026-02-18)
- The `Always Active`/`noTrigger` setting has been removed from settings UI.
- Default trigger remains `:e` (conflict-avoidance with upstream launcher behavior).
- Added configurable clipboard backend:
  - default: DMS clipboard command (`dms cl copy`)
  - fallback: `wl-copy` when DMS command is unavailable
- Added launcher paste hook support (`getPasteText`/`getPasteArgs`) so `Shift+Enter` performs direct paste from launcher selection.
- Trigger defaults and settings usage examples are aligned to `:e`.

## Critical Information for AI Agents

### ⚠️ DO NOT READ catalog.js
**IMPORTANT**: `catalog.js` is auto-generated and extremely large (1.7MB, 200k+ characters). Reading it will cause crashes. Use `grep` or other tools to search it if needed.

### Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│  Data Files (Plain Text)                            │
│  ├── data/emojis.txt          (emoji database)      │
│  ├── data/latin-extended.txt  (accented chars)      │
│  ├── data/math.txt            (unicode symbols)     │
│  └── data/nerdfont.txt        (nerd font glyphs)    │
└─────────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────────┐
│  Build Script                                        │
│  scripts/generate_catalog.py                        │
│  - Parses plain text files                          │
│  - Generates keywords                               │
│  - Adds language context for latin-extended         │
│  - Outputs JavaScript catalog                       │
└─────────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────────┐
│  Generated Catalog                                   │
│  catalog.js                                          │
│  - JavaScript library (.pragma library)             │
│  - Exports getEmojiEntries()                        │
│  - Exports getUnicodeEntries()                      │
│  - Exports getNerdFontEntries()                     │
│  - Exports getLatinExtendedEntries()                │
└─────────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────────┐
│  QML Components                                      │
│  ├── EmojiLauncher.qml         (main component)     │
│  ├── EmojiLauncherSettings.qml (settings UI)        │
│  └── plugin.json               (metadata)           │
│                                                      │
│  Imports catalog.js as CatalogData                  │
│  Calls getXXXEntries() in loadBundledData()         │
└─────────────────────────────────────────────────────┘
```

## File Structure

### Core Files
- **plugin.json** - Plugin metadata, version, trigger, capabilities
- **EmojiLauncher.qml** - Main launcher component (~130KB, 5200+ lines)
- **EmojiLauncherSettings.qml** - Settings UI component
- **catalog.js** - Generated character database (DO NOT READ DIRECTLY)

### Data Files (Plain Text)
Format: `[character] [DESCRIPTION]`

- **data/emojis.txt** - 900+ emoji entries with descriptions
- **data/latin-extended.txt** - 219 accented Latin characters
  - Organized by language sections (Spanish, French, German, etc.)
  - Comments like `# Spanish` are parsed for language keywords
- **data/math.txt** - Mathematical and general unicode symbols
- **data/nerdfont.txt** - Nerd Font glyphs (VS Code Codicons, Powerline)

### Build Scripts
- **scripts/generate_catalog.py** - Main build script
  - Parses all data/*.txt files
  - Generates keywords from character names
  - Adds language context for latin-extended.txt
  - Outputs catalog.js

## Key Concepts

### Character Entry Structure

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devnullvoid/dms-emoji-launcher](https://github.com/devnullvoid/dms-emoji-launcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
