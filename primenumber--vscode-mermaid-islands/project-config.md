---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- **Build**: `npm run compile` - Compiles TypeScript to JavaScript in the `out/` directory
- **Watch mode**: `npm run watch` - Automatically recompiles on changes
- **Lint**: `npm run lint` - Runs ESLint on the `src/` directory
- **Test**: `npm run test` - Runs tests using vscode-test framework
- **Pre-test**: `npm run pretest` - Compiles and lints before running tests
- **Package**: `npm run vscode:prepublish` - Prepares extension for publishing

## Dependencies

- **Core**: `puppeteer-core` for headless browser automation (requires system browser)
- **Rendering**: `mermaid` library loaded via CDN for diagram generation
- **Browser Detection**: Custom browser finder for Chrome/Edge/Chromium auto-detection
- **Development**: Standard VS Code extension toolchain with TypeScript and ESLint

## Project Architecture

This is a VS Code extension that renders Mermaid diagrams and SVG graphics as overlays within code comments. The extension uses VS Code's text decoration API with advanced features including dynamic sizing, scrolling support, and smart clipping.

The codebase is organized into modular components for maintainability and separation of concerns:

### Core Components

**Extension Entry Point** (`src/extension.ts`):
- Clean 32-line entry point that coordinates all components
- Main `activate()` function initializes the extension and event listeners
- Sets up event handlers for editor changes, text changes, selection changes, and scrolling
- Manages the lifecycle of the MermaidDecorationProvider

**Mermaid Decoration Provider** (`src/mermaidDecorationProvider.ts`):
- Main orchestrator that coordinates SVG rendering and decoration management
- Handles the complete workflow from parsing blocks to applying decorations
- Manages visible range calculations and decoration updates

**Mermaid Parser** (`src/mermaidParser.ts`):
- Detects both Mermaid and SVG code blocks using regex patterns:
  - Mermaid: `/\/\/\s*mermaid\s*\n([\s\S]*?)\/\/\s*end-mermaid/g`
  - SVG: `/\/\/\s*svg\s*\n([\s\S]*?)\/\/\s*end-svg/g`
- Cleans code by removing comment prefixes (`//`, `#`, `--`)
- Filters blocks where cursor is active (for editing mode)
- Returns blocks with type information ('mermaid' | 'svg')

**SVG Renderer** (`src/svgRenderer.ts`):
- Handles both Mermaid diagram generation and raw SVG processing
- Uses Puppeteer-core with automatic browser detection (Chrome/Edge/Chromium)
- Implements comprehensive caching with theme-specific cache keys (Light/Dark/HighContrast/HighContrastLight)
- Three-tier browser initialization: user-configured → auto-detected → bundled
- Handles error cases with theme-aware fallback error SVGs optimized for accessibility
- Manages browser instance lifecycle and provides helpful error messages

**Browser Finder** (`src/browserFinder.ts`):
- Automatically detects system browsers across Windows, macOS, and Linux
- Searches common installation paths for Chrome, Edge, and Chromium
- Provides fallback browser discovery when bundled Chromium unavailable

**Decoration Manager** (`src/decorationManager.ts`):
- Creates and manages VS Code text decorations
- Handles decoration caching to prevent unnecessary recreation
- Manages decoration lifecycle and cleanup

**Editor Utilities** (`src/editorUtils.ts`):
- Calculates block heights based on VS Code editor settings
- Determines visible portions of blocks during scrolling
- Handles precise clipping calculations for partial line visibility

**Type Definitions** (`src/types.ts`):
- TypeScript interfaces for MermaidBlock (with type: 'mermaid' | 'svg'), SvgRenderResult, and configuration
- Ensures type safety throughout the codebase
- Supports both Mermaid and SVG block types

**Constants** (`src/constants.ts`):
- Centralized configuration including regex patterns, cache limits, timeouts
- Mermaid theme configurations for all VS Code theme types (Light, Dark, HighContrast, HighContrastLight)
- Default dimensions and Puppeteer arguments

**Theme Utilities** (`src/themeUtils.ts`):
- Detects current VS Code theme kind and provides appropriate Mermaid configuration
- Supports all ColorThemeKind values: Light, Dark, HighContrast, and HighContrastLight
- Provides theme-aware color schemes optimized for accessibility and visibility

### Advanced Features

**Dynamic Height Matching**: Decorations automatically scale to match comment block height by reading VS Code editor settings (`editor.fontSize`, `editor.lineHeight`)

**Aspect Ratio Preservation**: Image width calculated based on SVG dimensions to maintain proportions

**Smart Clipping**: When scrolled, images clip from the top instead of scaling, showing the correct portion of the diagram

**Visible Range Detection**: Only renders decorations for visible portions of comment blocks to optimize performance

**Real-time Updates**: Responds to scrolling, text changes, cursor movement, and file switching

**Performance Optimizations**:
- **Browser Instance Reuse**: Single browser shared across all renders
- **SVG Result Caching**: LRU cache (100 items) with theme-specific keys for instant duplicate renders

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [primenumber/vscode-mermaid-islands](https://github.com/primenumber/vscode-mermaid-islands) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
