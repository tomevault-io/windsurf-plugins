---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## CRITICAL: Data Preservation Rules

**USER DATA IS SACRED. NEVER make changes that could delete or corrupt user prompts.**

This app stores user prompts in:
- **Local**: `~/Library/Application Support/com.promptlight.app/prompts/`
- **Cloud**: Firestore (when sync is enabled)

### DO NOT:
1. **Delete or modify** files in the prompts directory during development
2. **Modify sync logic** (src-tauri/src/data/sync.rs) without extreme care
3. **Change the data store** implementation without backup verification
4. **Trigger full sync operations** (`sync_to_cloud`, `sync_from_cloud`) during dev testing
5. **Reset Zustand stores** in ways that could trigger empty-state syncs

### Sync Architecture Warning:
- `sync_from_firestore()` REPLACES local data with cloud data
- `sync_to_firestore()` REPLACES cloud data with local data
- If either side has empty data, it will wipe out the other
- Auth flow (`set_sync_auth`) auto-triggers `sync_from_firestore`

### Before Modifying Data Layer:
1. Back up `~/Library/Application Support/com.promptlight.app/`
2. Test with a separate test account, not production data
3. Verify data integrity after changes

### Historical Data Loss Incidents:
- Changes to caching layer have caused data loss during dev mode
- Hot Module Reload (HMR) can reset stores, potentially triggering problematic syncs

## Build & Development Commands

```bash
# Development
npm run dev                # Start dev server with hot reload (full Tauri app)
npm run dev:vite           # Start Vite only (frontend, no Tauri)

# Build & Install
npm run build              # TypeScript check + Vite build (frontend only)
npm run install:local      # Build release and install to /Applications (macOS)

# CRITICAL: Building release DMGs with OAuth support
# Must source .env.local BEFORE building to embed Google OAuth credentials
source .env.local && npm run tauri build -- --target universal-apple-darwin
# If credentials aren't embedded, Sign in with Google will silently fail

# Unit Testing
npm run test               # Run all unit tests once
npm run test:watch         # Run tests in watch mode
npm run test:coverage      # Run tests with coverage report

# Run a single test file
npx vitest run src/__tests__/stores/editorStore.test.ts

# Run tests matching a pattern
npx vitest run -t "should load prompts"

# E2E Testing (Playwright)
npx playwright test        # Run all E2E tests
npx playwright test --headed  # Run with browser visible
npx playwright test src/__tests__/e2e/launcher.spec.ts  # Run specific file

# Linting
npm run lint               # Run ESLint
npm run lint:fix           # Run ESLint with auto-fix

# Full validation (run before committing)
npm run check              # Runs: lint → build → test

# Rust tests
cd src-tauri && cargo test
```

## Code Quality Requirements

**Before committing any code, run `npm run check`** which validates:
1. ESLint passes (warnings ok, errors fail)
2. TypeScript compiles without errors
3. All tests pass

**TypeScript Strictness**: The build enforces `noUnusedLocals` and `noUnusedParameters`. Unused code will fail the build.

**ESLint Rules**: Key rules enforced:
- No unused variables (prefix with `_` to ignore)
- `console.log` discouraged (use `console.error` or `console.warn` for actual issues)
- Prefer `const` over `let`
- Use `===` instead of `==`

## macOS Performance Critical

**NEVER use AppleScript on macOS.** AppleScript has severe performance penalties (~200ms+ per call) and is unreliable.

Instead, use native APIs:
- **App focus tracking**: `NSRunningApplication` via objc bindings (see `src-tauri/src/os/platform/macos.rs`)
- **Input simulation**: `CGEvent` for keyboard events
- **Window positioning**: `NSScreen` native API (see `src-tauri/src/os/focus.rs`)

The pattern to follow is in `src-tauri/src/os/platform/macos.rs` - direct Cocoa/Core Graphics calls via the `objc` crate.

## Architecture Overview

This is a Tauri 2.0 desktop app (Rust backend + React frontend) - a Spotlight-style prompt launcher.

### Backend Layers (src-tauri/src/)

```
os/                    # Platform-specific OS interactions
├── platform/          # Per-OS implementations (macos.rs, windows.rs, linux.rs)
├── paste.rs           # Clipboard + paste-back to previous app
├── focus.rs           # Screen/window focus detection
├── previous_app.rs    # Track app that was active before launcher
└── window.rs          # Window management

data/                  # Data persistence (JSON files in ~/Library/Application Support)
├── index.rs           # Prompt index + folder management
├── prompt.rs          # Individual prompt CRUD
├── search.rs          # Fuzzy search across prompts
└── stats.rs           # Usage tracking
```

### Frontend Layers (src/)

```
components/
├── launcher/          # Spotlight-style popup (SearchBar, ResultsList, etc.)
└── editor/            # Full prompt editor window (Sidebar, PromptEditor, etc.)

stores/                # Zustand state management
├── launcherStore.ts   # Launcher window state
├── launcherCacheStore.ts  # Search result caching
└── editorStore.ts     # Editor window state


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bladnman/promptlight](https://github.com/bladnman/promptlight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
