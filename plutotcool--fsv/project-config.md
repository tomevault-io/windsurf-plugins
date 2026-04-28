---
trigger: always_on
description: `@plutotcool/fsv` is a TypeScript library that defines and implements the **FSV (Fast Scrubbing Video)** format — a custom binary video format optimized for frame-accurate scrubbing on the web, with optional alpha-channel support.
---

# Copilot Instructions for `@plutotcool/fsv`

## Project Overview

`@plutotcool/fsv` is a TypeScript library that defines and implements the **FSV (Fast Scrubbing Video)** format — a custom binary video format optimized for frame-accurate scrubbing on the web, with optional alpha-channel support.

The library ships two distinct APIs:

| API | Environment | Purpose |
|-----|-------------|---------|
| **Conversion** | Node.js | Converts any video file to `.fsv` using ffmpeg (`node-av`) and WebCodecs bindings (`@napi-rs/webcodecs`) |
| **Decoding / Rendering** | Browser | Loads, demuxes, decodes, and WebGL2-renders `.fsv` files using native browser WebCodecs and WebGL2 APIs |

The package is published to the GitHub Packages registry (`https://npm.pkg.github.com`) under the `@plutotcool` scope.

---

## Repository Layout

```
fsv/
├── src/
│   ├── index.ts          # Browser-side public exports (Renderer, Decoder, Demuxer, FSV types)
│   ├── cli/
│   │   ├── index.ts      # CLI entry point (#!/usr/bin/env node, runs citty main)
│   │   ├── main.ts       # CLI main command definition (citty)
│   │   └── convert.ts    # `fsv convert` sub-command
│   └── core/
│       ├── FSV.ts        # FSV & FSVTrack TypeScript interfaces
│       ├── Video.ts      # Shared Video interface (seek/progress/set/width/height/duration/length)
│       ├── Manifest.ts   # Binary manifest serialization/deserialization (compact flat number arrays)
│       ├── Converter.ts  # Node.js-only: ffmpeg transcode → FSV (uses node-av + FSVMuxer)
│       ├── Muxer.ts      # Node.js-only: pack encoded mp4/webm data into .fsv binary
│       ├── Demuxer.ts    # Browser-compatible: unpack .fsv binary into FSV objects (sync + streaming)
│       ├── Decoder.ts    # Browser: coordinate color+alpha TrackDecoders, invoke user callback
│       ├── TrackDecoder.ts # Browser: WebCodecs VideoDecoder wrapper for a single FSV track
│       ├── Renderer.ts   # Browser: WebGL2 renderer using decoded VideoFrames
│       ├── Logger.ts     # Thin consola wrapper for CLI/Converter logging
│       └── shaders/
│           ├── renderer.vert  # GLSL vertex shader (imported as string via tsdown loader)
│           └── renderer.frag  # GLSL fragment shader (#define ALPHA 1 injected for alpha videos)
├── @types/
│   └── shaders.d.ts      # Module declarations for *.vert and *.frag text imports
├── demo/                 # Vite demo app (browser-side usage showcase)
├── .github/
│   └── workflows/
│       ├── ci.yml        # PR checks: typecheck + build
│       └── release.yml   # Push to main: build + changelogen release + npm publish
├── tsconfig.json         # strict, ESNext, Bundler resolution, path aliases (~/→src/, ~~/→root/)
├── tsdown.config.ts      # Build config: unbundle, ESM+CJS, dts, .vert/.frag as text
├── package.json          # Scripts, exports map, bin entry, peerDeps, publishConfig
└── pnpm-workspace.yaml   # allowBuilds for native node-av modules
```

---

## Code Style & Conventions

- **Language**: TypeScript (strict mode, `strictNullChecks`, `noUnusedLocals`)
- **Indentation**: 2 spaces (no tabs)
- **Quotes**: Single quotes
- **Semicolons**: None at end of statements
- **Type imports**: Use `import type` for type-only imports
- **JSDoc**: Required on all public API members (classes, methods, interfaces, exported functions)
- **Path aliases**: `~/` maps to `src/`, `~~/` maps to the repo root (tsconfig `paths`)

---

## Package Manager & Tooling

| Tool | Purpose |
|------|---------|
| **pnpm** | Package manager (managed via `corepack enable`) |
| **tsdown** | Build tool — outputs unbundled ESM (`.mjs`) + CJS (`.cjs`) + type declarations (`.d.ts`) |
| **tsc** | Type checking only (no emit, `noEmit: true`) |
| **tsx** | Runs the CLI directly from TypeScript source in development (`pnpm fsv`) |
| **Vite** | Demo app dev server and build |
| **changelogen** | Changelog generation + npm release (`pnpm release`) |

---

## Key Commands

```shell
# Install (IMPORTANT: do not use --ignore-scripts; ffmpeg bindings require lifecycle scripts)
corepack enable
pnpm install

# Type-check (runs tsc --noEmit)
pnpm typecheck

# Build the package (tsdown → dist/)
pnpm build

# Run the CLI directly via tsx (development, no build required)
pnpm fsv convert --help
pnpm fsv convert input.webm output.fsv --alpha --crf 20

# Run the demo app locally
pnpm dev

# Build the demo
pnpm build:demo

# Create a release (bump version, update changelog, push tag, publish to GitHub Packages)
pnpm release
```

---

## CI/CD Workflows

- **`ci.yml`** — Triggered on PRs to `main`. Runs two parallel jobs:
  1. **Typecheck** (`pnpm typecheck`)
  2. **Build** (`pnpm build`)
- **`release.yml`** — Triggered on push to `main`. Builds the package and runs `pnpm release` (changelogen) to publish to `https://npm.pkg.github.com`.

Always ensure both `pnpm typecheck` and `pnpm build` pass before merging.

---

## Architecture Deep-Dive

### The FSV Binary Format

Non-alpha videos:
```
[4 bytes: 0x00000000 (null)] [4 bytes: manifest byte length] [manifest JSON] [encoded video data]
```

Alpha videos (color + alpha tracks):
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [plutotcool/fsv](https://github.com/plutotcool/fsv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
