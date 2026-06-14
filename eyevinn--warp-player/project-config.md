---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development

```bash
# Install dependencies
npm install

# Start development server (HTTPS on port 8080)
npm start
# or
npm run dev

# Build for production
npm run build

# Run tests
npm test

# Run a specific test
npx jest src/buffer/mediaBuffer.test.ts

# Lint code
npm run lint

# Fix linting issues
npm run lint:fix

# Check code styling
npm run pretty

# Fix code formatting issues
npx prettier --write <files>
# or for all files:
npx prettier --write --ignore-unknown .

# Type checking
npm run typecheck
```

### Git Hooks

The project has pre-push hooks that automatically run before pushing:

- TypeScript type checking
- ESLint
- Jest tests

These ensure code quality before changes are pushed to the repository.

## Project Overview

WARP Player is a browser-based TypeScript implementation of a media player,
using the MOQ Transport protocol via WebTransport. It supports MOQ Transport
draft-14 and draft-16 (negotiated through WebTransport ALPN, can be forced
from the UI) and uses the MSF/CMSF catalog format
(draft-ietf-moq-msf-01 / draft-ietf-moq-cmsf-00) to discover media tracks.

Playback runs through one of two interchangeable render pipelines selected
per session:

- **MSE / CMAF** for `packaging: "cmaf"` and `packaging: "locmaf"` tracks,
  with optional EME for encrypted content (Widevine, PlayReady, FairPlay,
  ClearKey). LOCMAF objects are decompressed into standard CMAF before
  being appended to the `SourceBuffer`.
- **WebCodecs / LOC** for `packaging: "loc"` tracks (draft-mzanaty-moq-loc),
  clear content only, supporting AVC and HEVC video plus AAC and Opus audio.

Both pipelines implement a common `IPlaybackPipeline` interface so the
buffer-control loop, latency reporting, mute toggle, and namespace selector
work uniformly regardless of which engine is active.

### Project Structure

The project follows the Eyevinn TypeScript project template structure:

```
warp-player/
├── src/
│   ├── transport/        # MOQ protocol implementation (draft-14 / draft-16)
│   │   ├── client.ts     # WebTransport client implementation
│   │   ├── setup.ts      # Setup message handling
│   │   ├── tracks.ts     # Track subscription and management
│   │   ├── control.ts    # Control stream handling
│   │   └── version.ts    # Draft version constants and ALPN strings
│   ├── buffer/           # CMAF segment buffering for the MSE pipeline
│   │   ├── mediaBuffer.ts         # CMAF segment parsing
│   │   └── mediaSegmentBuffer.ts  # Buffer management for MSE
│   ├── loc/              # LOC payload helpers for the WebCodecs pipeline
│   │   ├── avc.ts        # AVC NALU walker, AVCDecoderConfigurationRecord
│   │   ├── hevc.ts       # HEVC NALU walker, HEVCDecoderConfigurationRecord
│   │   ├── aac.ts        # AAC AudioSpecificConfig from catalog metadata
│   │   ├── opus.ts       # Opus ID-header (OpusHead) builder
│   │   └── extensions.ts # LOC extension-header parsing (capture timestamps)
│   ├── locmaf/           # LOCMAF (compressed CMAF) helpers for MSE pipeline
│   │   └── locmaf.ts     # Parse LOCMAF init/moof/delta-moof, rebuild CMAF
│   ├── pipeline/         # Pluggable render pipelines
│   │   ├── index.ts                # IPlaybackPipeline + capability matrix
│   │   ├── msePipeline.ts          # MSE/CMAF pipeline (with optional EME)
│   │   └── webcodecsLocPipeline.ts # WebCodecs/LOC pipeline (clear only)
│   ├── warpcatalog.ts    # MSF/CMSF catalog types and parsing
│   ├── player.ts         # Core player: catalog → tracks → pipeline
│   ├── browser.ts        # Browser entry point and UI handling
│   └── index.html        # HTML template and UI components
├── FINGERPRINT.md        # Documentation for fingerprint feature
├── tsconfig.json         # TypeScript configuration
├── tsconfig.base.json    # Base TypeScript configuration
├── webpack.config.js     # Webpack configuration
├── jest.config.js        # Jest test configuration
├── package.json          # Project dependencies and scripts
└── .github/              # GitHub Actions workflows
    └── workflows/
        ├── ci.yml              # Main CI workflow (lint, test, build)
        ├── commitlint.yml      # Commit message linting
        └── dependency-review.yml # Security checks for dependencies
```

### Core Architecture

The transport used is MOQ Transport, draft-14 or draft-16 (auto-negotiated
via WebTransport ALPN strings `moq-00` and `moqt-16`).

For the catalog, the specification used is MSF (draft-ietf-moq-msf-01)
with CMSF (draft-ietf-moq-cmsf-00) for CMAF packaging. LOC packaging
follows draft-mzanaty-moq-loc.

The codebase is organized into several key modules:

1. **Transport Layer**:
   - Located in `src/transport/`
   - Handles WebTransport connection and MOQ protocol implementation
   - Manages bidirectional control streams and unidirectional data streams
   - Implements client-server setup messaging, track subscription, and data reception

2. **Catalog Layer**:
   - Located in `src/warpcatalog.ts`
   - Parses MSF/CMSF catalogs, including delta updates, content protection,
     and namespace inheritance for tracks that omit an explicit namespace


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Eyevinn/warp-player](https://github.com/Eyevinn/warp-player) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-14 -->
