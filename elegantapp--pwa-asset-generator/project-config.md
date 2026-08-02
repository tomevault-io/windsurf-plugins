---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

pwa-asset-generator is a CLI tool and JavaScript module that automates PWA asset generation. It generates icon and splash screen images for Progressive Web Apps, automatically updating manifest.json and index.html files according to Web App Manifest specs and Apple Human Interface guidelines.

The tool uses Puppeteer to control a Chrome browser as a canvas, rendering images at various resolutions for different devices. It can scrape Apple's Human Interface guidelines website to get the latest device specifications or fall back to static data.

## Essential Commands

### Development workflow
```bash
npm install              # Install dependencies
npm run build            # Build TypeScript to dist/ (also copies JSON files)
npm run start            # Watch mode for TypeScript compilation
npm run lint             # Run ESLint
npm run lint:fix         # Fix ESLint issues
npm run prettier         # Check code formatting
npm run prettier:fix     # Auto-format code
npm run tsc              # Type check without emitting files
```

### Testing
```bash
npm test                                  # Run all tests with Vitest
npm run test:update                       # Update all snapshots (visual + regular)
npm run test:update:snapshots             # Update regular snapshots only
npm run test:update:visuals               # Update all visual test snapshots
npm run chromium                          # Install Chromium (required for tests)
npm run test:concurrency                  # Concurrency stress test (see below)
```

Note: Visual tests generate actual images in `src/__snapshots__/visual/` and compare them. When updating visual tests, ensure you're not accidentally breaking existing behavior.

### Concurrency stress test

`scripts/concurrency-stress-test.mjs` exercises the `saveImages` worker pool with 400+ images across simulated hardware profiles. It prints the computed concurrency for every profile, then runs a real end-to-end test.

```bash
# Run against real hardware (auto-detected CPU count and free memory)
npm run test:concurrency

# Simulate a specific hardware profile via env vars
PAG_SIMULATE_CPU_COUNT=1 PAG_SIMULATE_FREE_MEM_MB=512 npm run test:concurrency
PAG_SIMULATE_CPU_COUNT=2 PAG_SIMULATE_FREE_MEM_MB=2048 npm run test:concurrency
```

`PAG_SIMULATE_CPU_COUNT` and `PAG_SIMULATE_FREE_MEM_MB` override the values that `getOptimalConcurrency` reads from `os.cpus()` and `os.freemem()`, letting you reproduce any hardware scenario locally without Docker.

### Running the CLI locally
```bash
npm run build                             # Build first
node bin/cli.js <source> <output>         # Run CLI directly
# Or install globally for testing:
npm i . -g
pwa-asset-generator <source> <output>
```

### Updating fallback data
```bash
npm run update           # Scrape Apple HIG and update apple-fallback-data.json
```

This scrapes the latest device specs from Apple's website and updates `src/config/apple-fallback-data.json`.

## Architecture

### Core flow
1. **CLI Entry** (`src/cli.ts`): Parses arguments using meow, validates flags
2. **Main Function** (`src/main.ts`): Orchestrates the generation process via `generateImages()`
3. **Image Generation** (`src/helpers/puppets.ts`): Controls Puppeteer to generate images
4. **Meta Generation** (`src/helpers/meta.ts`): Generates HTML meta tags and manifest.json content
5. **File Operations** (`src/helpers/file.ts`): Handles file I/O and path resolution

### Key components

**Puppets helper** (`src/helpers/puppets.ts`):
- Launches Puppeteer browser
- Scrapes Apple HIG website for device specs (or uses fallback data)
- Creates a shell HTML page as an "art board" for image rendering
- Takes screenshots at various resolutions
- Generates icons (manifest, apple-touch, favicon, mstile) and splash screens

**Meta helper** (`src/helpers/meta.ts`):
- Generates HTML meta tags for iOS splash screens and icons
- Generates manifest.json icon entries
- Updates existing manifest.json and index.html files using Cheerio
- Formats output using the `pretty` library

**Browser helper** (`src/helpers/browser.ts`):
- Manages Puppeteer browser lifecycle
- Handles chromium installation check
- Uses puppeteer-core to avoid bundling Chromium

**Flag/Options handling** (`src/helpers/flags.ts`):
- Normalizes CLI flags into internal options
- Handles mutually exclusive flag pairs (e.g., `splashOnly`/`iconOnly`)
- Validates sandbox options

### File structure
```
src/
├── cli.ts                    # CLI entry point
├── main.ts                   # Main generateImages() function
├── config/
│   ├── constants.ts          # All constants, flag definitions
│   └── apple-fallback-data.json  # Static device specs
├── models/                   # TypeScript type definitions
│   ├── image.ts
│   ├── meta.ts
│   ├── options.ts
│   ├── result.ts
│   ├── spec.ts
│   └── logger.ts
└── helpers/                  # Core logic modules
    ├── puppets.ts           # Puppeteer orchestration
    ├── meta.ts              # HTML/manifest generation
    ├── browser.ts           # Browser management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elegantapp/pwa-asset-generator](https://github.com/elegantapp/pwa-asset-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
