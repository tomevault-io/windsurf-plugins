---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`vite-plugin-webfont-dl` is a Vite plugin that downloads webfonts from third-party providers (Google Fonts, Bunny Fonts, Fontshare, jsDelivr, rsms.me) and self-hosts them. This eliminates render-blocking external requests, improves page load performance, and protects user privacy by preventing third-party tracking.

**Core functionality:**
- Extracts webfont URLs from HTML `<link>` tags, plugin config, and CSS `@import` statements
- Downloads webfont CSS and parses `@font-face` definitions
- Downloads font files (woff2, woff, ttf, otf, eot, svg)
- Transforms CSS to reference local paths or embed fonts as base64
- Injects fonts into HTML as `<style>` tags or external CSS files
- Persistent file caching for offline development

## Commands

**Build:**
```bash
npm run build          # Production build with tsup (minified CJS + ESM + types)
npm start              # Watch mode for development
```

**Lint:**
```bash
npm run lint           # TypeScript check + ESLint (read-only)
npm run lint:fix       # ESLint with auto-fix
```

**Test:**
```bash
npm test               # Run all Vitest tests
```

## Architecture Overview

### Entry Point: src/index.ts

The main plugin file exports a Vite Plugin that implements four hooks:

1. **`configResolved`**: Captures Vite config (base URL, assets directory, minify settings, logger)
2. **`configureServer`**: Sets up dev server middleware for serving fonts dynamically
3. **`transformIndexHtml`**: Processes HTML files to extract and inject font references
4. **`generateBundle`**: Main build-time processing (download, parse, transform, inject)

**Critical detail**: The plugin has two completely different execution flows:
- **Build mode**: Pre-downloads all fonts at bundle generation time
- **Dev server mode**: Serves fonts on-demand via middleware, lazy-loading as needed

### Core Orchestrator: src/webfont-download.ts

The `WebfontDownload` class is the central coordinator that:
- Manages three webfont URL sources:
  1. **Plugin config**: URLs passed to the plugin constructor
  2. **HTML extraction**: `<link>` tags parsed from HTML files
  3. **CSS extraction**: `@import` statements found in bundled CSS
- Coordinates all components in sequence
- Maintains the font collection (Map<filename, Font>)
- Handles both dev server middleware and build-time processing
- Generates cache hit statistics for performance transparency

### Component Architecture (src/components/)

The plugin uses a component-based architecture with single-responsibility classes:

#### css-loader.ts
**Purpose**: Downloads webfont CSS from providers and normalizes URLs

**Key implementation details:**
- Uses FileCache for persistent storage
- **URL normalization edge cases**:
  - Protocol-relative URLs (`//fonts.googleapis.com/...`) → prefixed with `https:`
  - Relative URLs (`../fonts/`, `./fonts/`, `fonts/`) → resolved to absolute URLs using CSS file's origin
  - Fully-qualified URLs → used as-is
- Flash logging shows download progress in terminal

#### css-parser.ts
**Purpose**: Parses CSS to extract font URLs and `@font-face` definitions

**Key implementation details:**
- **Two regex patterns for font URLs**:
  1. Standard font URLs: `https://fonts.gstatic.com/.../font.woff2`
  2. Google Fonts Kit URLs: `https://fonts.gstatic.com/l/font?kit=...` (special format)
- **Google Fonts Kit handling**:
  - Extracts `kit` parameter as filename
  - If kit string >50 chars, generates SHA1 hash as filename
  - Always uses `.woff2` extension
- **Subset filtering**:
  - Google Fonts CSS includes comments like `/* latin */` before `@font-face` blocks
  - When `subsetsAllowed` option is set, only fonts with matching comment tags are included
  - Reduces font files downloaded by filtering out unwanted language subsets
- **`parseBundleCss` method** (separate from `parse`):
  - Scans user's bundled CSS for webfont references
  - Extracts both `@font-face` definitions and `@import` statements
  - Only processes fonts from whitelisted providers (security measure)
  - Handles three `@import` syntax variants:
    - `@import url('...');`
    - `@import"...";`
    - `@import '...';`
- **Provider whitelist**: Google Fonts, Bunny Fonts, Fontshare, Google Fonts Static
- Returns deduplicated fonts via Map (filename as key ensures uniqueness)

#### css-transformer.ts
**Purpose**: Replaces remote font URLs with local paths or embeds as base64

**Key implementation details:**
- **Two transformation modes**:
  1. **URL replacement**: Replaces `https://fonts.gstatic.com/.../font.woff2` with `/assets/font.woff2`
  2. **Base64 embedding**: Converts font to `data:font/woff2;base64,...`
- **MIME type mapping**: Each font extension has correct MIME type (woff2→font/woff2, svg→image/svg+xml, etc.)
- **Regex escaping**: Font URLs are escaped before regex replacement to handle special characters in query strings
- **Minification**: Uses CleanCSS library, only applied in build mode when `minifyCss: true`

#### css-injector.ts
**Purpose**: Injects transformed CSS into HTML

**Key implementation details:**
- **Three injection modes**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [feat-agency/vite-plugin-webfont-dl](https://github.com/feat-agency/vite-plugin-webfont-dl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
