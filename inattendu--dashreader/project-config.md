---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
# Development with watch mode
npm run dev

# Production build (runs TypeScript check + esbuild)
npm run build

# Version bump (updates manifest.json and versions.json)
npm run version
```

**Important**: The build output `main.js` MUST be tracked in git for Obsidian plugin releases. It's intentionally not in .gitignore.

## Architecture Overview

### Plugin Structure

DashReader is an Obsidian plugin implementing RSVP (Rapid Serial Visual Presentation) speed reading. The architecture follows a clear separation:

**Core Architecture** (6 main files):

- **main.ts** - Plugin entry point, registers commands, ribbon icons, and manages view lifecycle
- **src/rsvp-view.ts** - UI layer (ItemView), handles user interactions, cursor tracking, and display
- **src/rsvp-engine.ts** - Core reading engine, controls timing, word iteration, and micropause logic
- **src/markdown-parser.ts** - Transforms Markdown to plain text while marking headings with `[H1]`, `[H2]` etc.
- **src/settings.ts** - Settings UI using Obsidian's PluginSettingTab
- **src/types.ts** - Shared interfaces and default settings

**Support Modules** (extracted during refactoring Phase 2):

- **src/constants.ts** - Centralized CSS classes, timing values, limits, and magic numbers
- **src/logger.ts** - Centralized logging with DashReader: prefix
- **src/hotkey-handler.ts** - Keyboard event handling (Shift+Space, navigation hotkeys)
- **src/word-display.ts** - Word rendering logic with heading/callout support
- **src/dom-registry.ts** - DOM element management and lifecycle
- **src/view-state.ts** - Reactive state management with change tracking
- **src/breadcrumb-manager.ts** - Breadcrumb navigation UI and logic
- **src/minimap-manager.ts** - Vertical minimap visualization
- **src/menu-builder.ts** - Dropdown menu creation for navigation
- **src/auto-load-manager.ts** - Auto-load text from editor on file-open/selection
- **src/ui-builders.ts** - UI component builders (buttons, sliders, toggles)

**Services** (business logic extraction):

- **src/services/timeout-manager.ts** - Timer management with cleanup
- **src/services/settings-validator.ts** - Settings validation and sanitization
- **src/services/micropause-service.ts** - Micropause calculation using Strategy Pattern
- **src/services/stats-formatter.ts** - Statistics formatting (time, WPM, progress)

### Key Architecture Patterns

**View-Engine Separation**: The view (`rsvp-view.ts`) owns the UI and event handling, while the engine (`rsvp-engine.ts`) owns reading logic and timing. They communicate via:
- View → Engine: `setText()`, `play()`, `pause()`, `updateSettings()`
- Engine → View: `onWordChange` callback, `onComplete` callback

**Cursor Position Tracking**: When loading text from the editor:
1. Parse Markdown FIRST to remove syntax (`markdown-parser.ts`)
2. Parse text up to cursor position separately
3. Count words in parsed text (not raw Markdown with frontmatter)
4. Pass word INDEX to engine, not character position

**Heading System**: Headings are marked during parsing (`# Title` → `[H1]Title`), then:
- View detects markers and displays with proportional font size (H1=1.5x, H2=1.3x, H3=1.2x, etc.)
- View adds visual separator lines before headings
- Engine applies longer micropauses (H1=2.0x, H2=1.8x, H3=1.5x, etc.)
- Headings extracted with full titles using line break markers (§§LINEBREAK§§)

**Breadcrumb Navigation System** (v1.4.0): Provides document structure awareness and navigation:

- **Display**: Single-line breadcrumb showing hierarchical path: 📑 H1 › H2 › H3 ▼
- **Extraction**: Engine's `extractHeadings()` collects all headings and callouts during `setText()`
  - Stops at §§LINEBREAK§§ markers to capture complete titles
  - Returns `HeadingInfo[]` with level, text, wordIndex, and optional calloutType
- **Context Building**: `getCurrentHeadingContext()` builds hierarchical breadcrumb
  - Filters headings up to current word index
  - Maintains heading stack, pops when level decreases
  - Returns `HeadingContext` with breadcrumb array and current heading
- **Navigation**: Click heading to jump, click ▼ dropdown for same-level navigation
  - `navigateToHeading(wordIndex)` preserves playback state
  - Dropdown menu shows all headings of same level with numbering
  - Menu created in `document.body` with fixed positioning for proper display
  - Centered under breadcrumb with viewport overflow protection
- **Initial Display**: Breadcrumb shown immediately on `loadText()`, not just during playback
- **Update Optimization**: Breadcrumb only redraws when heading context changes
  - `lastHeadingContext` property caches previous context
  - `hasHeadingContextChanged()` compares new vs old context
  - Prevents DOM recreation on every word, keeps dropdown clickable during reading

**Callout Support** (v1.4.0): Full integration with Obsidian callouts:
- Parser marks callouts: `> [!type] Title` → `[CALLOUT:type]Title`
- Treated as pseudo-headings (level=0) in breadcrumb hierarchy
- Display with icon prefix (📝 note, 💡 tip, ⚠️ warning, etc.)
- Visual separator and 1.2x font size during reading

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [inattendu/dashreader](https://github.com/inattendu/dashreader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
