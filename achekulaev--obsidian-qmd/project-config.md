---
trigger: always_on
description: This file provides context for AI assistants working on this project.
---

# CLAUDE.md - Project Context for AI Assistants

This file provides context for AI assistants working on this project.

## Rules

- Always run `npm run build && npm test && npm run lint` before committing code changes, unless the build/test/lint was already run and passed in the current session with no code changes since.
- When committing in external repos or cloned directories (e.g. scratchpad), always check `git config user.name` and `git config user.email` in the **main project repo** first and configure the same values in the external repo before committing.

## Project Overview

**obsidian-qmd** is an Obsidian plugin that integrates [QMD (Quick Markdown Search)](https://github.com/tobi/qmd) to provide semantic-first search in Obsidian vaults. It's a desktop-only plugin that uses QMD's vector search (semantic) as the default, with keyword (BM25) search as a fallback.

## What Has Been Implemented

### Project Structure (Complete)
```
obsidian-qmd/
├── src/
│   ├── main.ts           # ✅ Plugin entry point
│   ├── settings.ts       # ✅ Settings types and defaults
│   ├── qmd.ts            # ✅ QMD CLI wrapper with queue management + cancellation
│   ├── searchModal.ts    # ✅ Search modal UI (SuggestModal)
│   ├── searchPane.ts     # ✅ Optional sidebar search pane (ItemView)
│   ├── settingsTab.ts    # ✅ Settings UI tab
│   ├── settings.test.ts  # ✅ Tests for settings
│   ├── qmd.test.ts       # ✅ Tests for QMD wrapper
│   └── __mocks__/
│       └── obsidian.ts   # ✅ Mock for Obsidian API
├── manifest.json         # ✅ Obsidian plugin manifest
├── package.json          # ✅ Dependencies and scripts
├── tsconfig.json         # ✅ TypeScript configuration
├── esbuild.config.mjs    # ✅ Build configuration
├── jest.config.js        # ✅ Test configuration
├── .eslintrc.js          # ✅ Linting configuration
├── version-bump.mjs      # ✅ Version management script
├── versions.json         # ✅ Version history
├── styles.css            # ✅ Plugin styles
├── .github/
│   └── workflows/
│       └── release.yml   # ✅ GitHub Actions release workflow
├── .gitignore            # ✅ Git ignore rules
├── LICENSE               # ✅ MIT License
├── README.md             # ✅ User documentation
└── CONTRIBUTING.md       # ✅ Contributor guidelines
```

### Core Features Implemented
1. **QMD CLI Wrapper** (`src/qmd.ts`)
   - Command queue (only one QMD process at a time)
   - All QMD commands: status, collection add, update, embed, vsearch, search
   - **Search cancellation** - `abortSearch()` kills running QMD process
   - Proper error handling with typed errors
   - JSON output parsing with slug-to-file path resolution

2. **Main Plugin** (`src/main.ts`)
   - Settings load/save
   - Desktop-only detection
   - File watcher for auto-indexing and auto-embedding (debounced, only when changes detected)
   - Optional periodic updates (skipped when no pending changes)
   - All commands registered
   - Ribbon icon support
   - Search pane view registration
   - Auto-detection of QMD binary in common paths

3. **Search Modal** (`src/searchModal.ts`)
   - SuggestModal-based interface
   - Semantic-first with fallback logic
   - **1000ms trailing-edge debounce** - waits for user to stop typing
   - **Animated progress bar** - appears below search input when searching
   - **Cancellable search** - typing kills in-flight search process via searchId generation tracking
   - **Smart file matching** - matches QMD's slugified paths to actual files via title or slug
   - **Search mode pill** - semi-transparent purple pill inside input field showing "semantic" or "keyword (fallback)"
   - **Clean results** - titles only (no file paths), diff hunk headers stripped from snippets
   - **50-character input limit**
   - Result rendering with optional scores

4. **Search Pane** (`src/searchPane.ts`)
   - ItemView-based sidebar pane
   - Persistent search interface
   - Same search logic as modal

5. **Settings Tab** (`src/settingsTab.ts`)
   - All settings from the spec
   - Test QMD button
   - Diagnostic display
   - Action buttons (update index, generate embeddings, etc.)

### Build Status
- ✅ `npm install` - Dependencies installed
- ✅ `npm run build` - Builds successfully, produces `main.js`
- ✅ `npm test` - All 32 tests pass
- ✅ `npm run lint` - No lint errors

## What Was Fixed

### Jest Mocking Issue (Resolved)

The original problem was that `qmd.test.ts` used `jest.mock()` which gets hoisted, causing a "Cannot access before initialization" error.

**Solution:** Refactored `qmd.ts` to use dependency injection for the `execAsync` function:
- Added an optional `execAsync` parameter to the `QMDWrapper` constructor
- Tests inject a mock function directly instead of using `jest.mock()`
- This makes the code more testable and avoids Jest hoisting issues

### Lint Errors (Resolved)
- Removed unused imports (`WorkspaceLeaf`, `App`, `TFile`)
- Prefixed unused parameters with underscore (`_oldPath`, `_isFallback`)

### QMD CLI Integration Fixes
- **--index flag position**: Fixed to place `--index` before subcommand (global option)
- **Collection detection**: Fixed to parse text output from `qmd collection list` instead of expecting errors
- **Status parsing**: Fixed to parse text output from `qmd status` (QMD doesn't output JSON)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [achekulaev/obsidian-qmd](https://github.com/achekulaev/obsidian-qmd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
