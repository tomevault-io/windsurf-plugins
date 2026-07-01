---
trigger: always_on
description: **Typewriter Mode** is an Obsidian community plugin that transforms the editor into a distraction-free writing environment. It adds typewriter scrolling, current-line highlighting, text dimming, writing focus mode, Hemingway mode, and more.
---

# Obsidian Typewriter Mode — Project Overview

**Typewriter Mode** is an Obsidian community plugin that transforms the editor into a distraction-free writing environment. It adds typewriter scrolling, current-line highlighting, text dimming, writing focus mode, Hemingway mode, and more.

- **Plugin ID**: `typewriter-mode`
- **Current version**: see `manifest.json`
- **Minimum Obsidian**: v0.15.0 (no legacy editor support)
- **Author**: Davis Riedel

---

## Tech Stack

| Tool | Role |
|------|------|
| **TypeScript** (strict) | Primary language |
| **Bun** | Runtime, package manager, bundler |
| **CodeMirror 6** | Editor manipulation (`@codemirror/state`, `@codemirror/view`) |
| **Obsidian API** | Plugin framework |
| **SCSS** (compiled via `grass`) | Styles |
| **Biome / Ultracite** | Linting and formatting |
| **Just** | Command runner |
| **Lefthook** | Git hooks |
| **Stylelint** | SCSS linting |

Path alias: `@/*` → `./src/*`

---

## Project Structure

```text
src/
├── main.ts                        # Plugin entry point (onload/onunload)
├── lib.ts                         # Core coordinator: features, commands, settings, CM6 extensions
├── capabilities/
│   ├── base/                      # Abstract base classes (Feature, Command, ToggleCommand, …)
│   ├── features/                  # 10 feature categories (typewriter, dimming, currentLine, …)
│   ├── commands/                  # 8 commands (toggles, move typewriter up/down, writing focus)
│   ├── constants.ts               # Enums and shared constants
│   └── settings.ts                # Settings schema, types, defaults, migration
├── cm6/
│   ├── plugin.ts                  # CM6 ViewPlugin: decorations, scroll/resize, cursor tracking
│   ├── typewriter-offset-calculator.ts
│   ├── highlight-sentence.ts
│   ├── selectors.ts
│   └── per-window-props.ts
├── components/
│   ├── settings-tab.ts            # Settings UI panel (features register their own controls)
│   └── update-modal.ts            # Update announcement dialog
└── styles/
    ├── editor/                    # Editor-specific SCSS
    └── ui/                        # UI component SCSS
scripts/                           # Bun build/release scripts
test-vault/                        # Local Obsidian vault for manual testing
dist/                              # Compiled output (main.js, styles.css, manifest.json)
```

---

## Architecture

### Feature & Command System

**`TypewriterModeLib`** (`lib.ts`) is the central coordinator. It loads features and commands, manages CM6 extensions, handles settings persistence, and coordinates per-window CSS variables and body classes.

**Feature hierarchy:**

```text
Loadable → Feature (typed settings path) → FeatureToggle (CSS class toggle)
                                               └── Concrete features
```

Each feature lives in `src/capabilities/features/` and self-registers its settings UI.

**Command hierarchy:**

```text
AbstractCommand → Command | EditorCommand | ToggleCommand
```

### Settings System

- Nested, typed settings object with dotted-path access (`"typewriter.isTypewriterScrollEnabled"`)
- `getSettingByPath()` / `setSettingByPath()` for compile-time-safe access
- `applyStartupMigrations()` migrates legacy flat format (pre-v1.2.0) to the new grouped format
- Cursor positions per file are persisted in `data.json`

### CodeMirror 6 Integration

**`TypewriterModeCM6Plugin`** (`cm6/plugin.ts`) is a `ViewPlugin` that:

- Applies decorations for current-line highlighting and fading (not direct DOM manipulation)
- Calculates typewriter scroll offset
- Tracks cursor position changes for restoration
- Uses `ResizeObserver` for responsive updates
- Distinguishes user events from programmatic changes via transaction metadata
- Fires a custom `"moveByCommand"` event for command-driven cursor movement

### Plugin Lifecycle

1. `Plugin.onload()` → `lib.load()` → loads settings, features, commands, CM6 extension
2. `onLayoutReady()` → announces update if version changed
3. Editor operations → CM6 plugin reacts to view updates
4. `Plugin.onunload()` → `lib.unload()` → disables all features

---

## Commands

| Just task | Description |
|-----------|-------------|
| `just dev` | Build + set up test vault (main dev workflow) |
| `just build` | Build plugin only |
| `just debug` | Build with debug statements enabled |
| `just typecheck` | TypeScript type check |
| `just lint` | Biome lint & format |
| `just stylelint` | SCSS linting |
| `just markdownlint` | Markdown linting |
| `just check` | All of the above |
| `just release` | Run checks then release (creates tag + GitHub release) |
| `just ci` | CI build (used in GitHub Actions) |

Output artifacts: `dist/main.js`, `dist/styles.css`, `dist/manifest.json`

---

## Key Patterns

- **Decorations over DOM**: CM6 decorations (`RangeSet`) for all visual effects — avoids direct DOM manipulation
- **Per-window state**: CSS variables and body classes tracked per editor window
- **Settings migration**: One-pass migration on load, version-gated for backward compatibility

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [davisriedel/obsidian-typewriter-mode](https://github.com/davisriedel/obsidian-typewriter-mode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
