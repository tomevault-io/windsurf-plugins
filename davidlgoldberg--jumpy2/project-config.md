---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Jumpy2 is a VS Code extension that creates dynamic hotkeys to jump around files across visible panes. Users press `shift+enter`, see labels appear at word positions, type 2 characters, and jump directly to that location. Works seamlessly with vim/neovim extensions.

**Supports both desktop and web environments** (github.dev, vscode.dev, Codespaces). See `BROWSER_SUPPORT.md` for web-specific details.

## Essential Commands

**IMPORTANT: Node Version Management**
This project requires Node 20.18.1. ALWAYS run `nvm use` before executing any npm commands to ensure the correct Node version is active.

### Development

```bash
# Switch to correct Node version (REQUIRED FIRST STEP)
nvm use

# Install dependencies
npm install

# Development watch mode (both TypeScript + esbuild)
npm run watch

# Build for production (desktop)
npm run compile

# Build for web/browser
npm run compile-web

# Type checking only
npm run check-types

# Lint code
npm run lint
```

### Testing

```bash
# Switch to correct Node version first
nvm use

# Run all tests (includes pretest compile and lint)
npm test

# Run tests without recompiling (faster iteration)
npm run test-no-compile

# Compile tests only
npm run compile-tests

# Watch tests (auto-recompile)
npm run watch-tests
```

### Web Development

```bash
# Build web bundle
npm run compile-web

# Watch mode for web development
npm run watch-web

# Test in local browser (Chromium)
npm run run-in-browser
```

### Publishing

```bash
# Publish to VS Code Marketplace (builds both desktop and web bundles)
npm run deploy:vsmarketplace

# Publish to Open VSX Registry (for Cursor, VSCodium)
npm run deploy:ovsx
```

## Architecture Overview

### Hybrid TypeScript + Elm Architecture

This extension uses a **unique hybrid architecture** that's critical to understand:

**TypeScript Layer** (`src/extension.ts`)
- Handles all VS Code API interactions
- Manages extension lifecycle, commands, and keybindings
- Coordinates label rendering via VS Code decorations
- Integrates telemetry and global state (achievements, career jumps)

**Elm State Machine** (`src/elm/StateMachineVSC.elm`)
- Pure functional state management for jump mode logic
- Processes key sequences and matches them to labels
- Compiled separately to JavaScript and imported by TypeScript
- Communicates via **ports** (Elm's FFI mechanism)

### Port-Based Communication Pattern

```
User presses key → TypeScript → stateMachine.ports.key.send(charCode)
                                        ↓
                              Elm processes state
                                        ↓
Elm port emits → validKeyEntered/labelJumped → TypeScript
                                        ↓
                     TypeScript updates UI/performs jump
```

**Outbound Ports (Elm → TypeScript):**
- `activeChanged`: Jump mode on/off
- `statusChanged`: Status bar text updates
- `validKeyEntered`: Filter labels after first character
- `labelJumped`: Execute jump to position

**Inbound Ports (TypeScript → Elm):**
- `getLabels`: Initialize with available labels
- `key`: User pressed a character
- `reset`: Backspace pressed, reset first character
- `exit`: Exit jump mode

### Label System Architecture

**Labeler Pattern** (`src/labelers/`)
- `words.ts`: Main `WordLabel` class implementing `Label` interface
- Finds word positions using configurable regex (`jumpy2.wordPattern`)
- Generates two-character key sequences from available character set
- Handles **wide characters** (CJK/emoji) with adjusted positioning

**Decoration System:**
- `wordDecorations.ts`: Two decoration types (base + checkered)
- Checkered mode alternates colors for adjacent labels
- Labels rendered as "after" decorations (pseudo-elements)
- Beacons (`wordBeacons.ts`): Animated red flash after jump

**Multi-Editor Support:**
Labels are generated across all `window.visibleTextEditors`, enabling jumps across split panes in a single activation.

### Dynamic Command Registration

The extension dynamically registers commands for every character:
- `jumpy2.a`, `jumpy2.b`, ..., `jumpy2.z`
- `jumpy2.A`, `jumpy2.B`, ..., `jumpy2.Z`

This happens at activation using `getAllKeys()` and iterating through character sets.

### Context-Based Keybindings

Uses `jumpy2.jump-mode` context (set via `setContext` command):
- When active: character keys route to Jumpy commands
- When inactive: normal editor keybindings apply
- Critical for vim/neovim integration and preventing key conflicts

### Debounced Exit Pattern

Multiple editor events trigger exit to handle user actions gracefully:
```typescript
window.onDidChangeActiveTextEditor
window.onDidChangeTextEditorSelection
window.onDidChangeVisibleTextEditors
// ... and others
```

All route through `_exitDebounced()` (350ms debounce, leading edge) to prevent premature mode clearing during normal operations.

## Working with Elm Code

### Critical: Separate Elm Compilation

Elm files must be compiled **before** esbuild can bundle the extension. The build system expects compiled Elm at `out/elm/StateMachineVSC.js`.

```bash
# Compile Elm manually if needed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DavidLGoldberg/jumpy2](https://github.com/DavidLGoldberg/jumpy2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
