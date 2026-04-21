---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

GPAC TwentySix is a fork of GPAC v2.4 stripped to BIFS (Binary Format for Scenes) core for interactive MPEG-4 content. The project targets a 70% code reduction (500K→150K LOC) and <5MB WASM output.

## Build Commands

```bash
# TypeScript packages (all)
bun run build              # Build all TS packages
bun run typecheck          # Type-check all TS packages
bun test                   # Run all tests

# Individual package builds
bun run build:shared       # Build shared types package
bun run build:player       # Build player SDK
bun run build:solidjs      # Build SolidJS component
bun run build:aot-compiler # Build AOT compiler
bun run build:compiler     # Build compiler service
bun run build:editor       # Build visual editor

# GPAC native build
bun run build:gpac         # Configure and build GPAC C source
bun run build:gpac:asan    # Build with AddressSanitizer
bun run clean:gpac         # Clean GPAC build
bun run test:gpac          # Verify GPAC build (runs MP4Box -version)

# WebAssembly build (requires Emscripten)
cd packages/gpac-core && ./build-wasm.sh         # Build WASM module
cd packages/gpac-core && ./build-wasm.sh clean   # Clean WASM artifacts

# Development
bun run dev:player         # Watch mode for player package
bun run dev:editor         # Run editor dev server (Vite)
bun run dev:compiler       # Run compiler service (watch mode)
bun run start:compiler     # Run compiler service (production)
bun run demo               # Build and serve player demo (in packages/player)

# Docker
bun run docker:build       # Build compiler Docker image
bun run docker:run         # Run compiler container
bun run docker:compose     # Run full stack (compiler + editor)

# Single test file
bun test packages/player/test/types.test.ts
bun test packages/aot-compiler/test/compiler.test.ts
```

## Architecture

```
gpac-twentysix/
├── packages/
│   ├── gpac-core/           # GPAC C source (autoconf/make build)
│   │   ├── src/bifs/        # BIFS encoder/decoder (core)
│   │   ├── src/scenegraph/  # Scene graph runtime
│   │   ├── src/isomedia/    # MP4 read/write
│   │   ├── build-wasm.sh    # Emscripten build script
│   │   └── dist-wasm/       # WASM output (gpac-core.js + .wasm)
│   │
│   ├── shared/              # @gpac-interactive/shared - Shared types & Zod schemas
│   │   └── src/schemas.ts   # Core types (Hotspot, Marker, SceneOp, etc.)
│   │
│   ├── player/              # @gpac-interactive/player - TypeScript SDK
│   │   ├── src/core/        # GPACInteractivePlayer class
│   │   ├── src/wasm/        # WASM bindings (BifsDecoder, MP4Reader)
│   │   ├── src/renderer/    # Canvas2DRenderer
│   │   ├── src/services/    # Effect services (WasmCore, CodecManager)
│   │   └── demo/            # Browser demo
│   │
│   ├── solidjs/             # @gpac-interactive/solidjs - SolidJS component
│   │   └── src/             # BIFSPlayer.tsx, hooks
│   │
│   ├── aot-compiler/        # @gpac-interactive/aot-compiler - AOT TypeScript compiler
│   │   ├── src/analyzer.ts  # Forbidden pattern detection
│   │   └── src/compiler.ts  # TS → SceneOp transformation
│   │
│   ├── compiler/            # @gpac-interactive/compiler - REST API service
│   │   ├── src/server.ts    # Bun.serve() endpoints
│   │   ├── src/mp4box.ts    # MP4Box subprocess wrapper
│   │   └── src/preview.ts   # GIF preview generation
│   │
│   └── editor/              # @gpac-interactive/editor - Visual editor (SolidJS)
│       ├── src/components/  # Toolbar, Timeline, Canvas, PropertiesPanel
│       └── src/stores/      # Project state management
│
├── test-videos/bifs/        # Test MP4 files with BIFS content
├── Dockerfile               # Compiler service Docker image
├── Dockerfile.editor        # Editor Docker image
├── docker-compose.yml       # Full stack orchestration
└── spec.md                  # Architecture document
```

## Key Technical Patterns

**Effect Library**: The player uses Effect for typed errors and dependency injection. Services (WasmCore, CodecManager) are defined as Effect layers. For Effect generators with class methods, use explicit type annotation:

```typescript
someMethod(): Effect.Effect<T, E> {
  const self: ClassName = this;  // Required for correct 'this' binding
  return Effect.gen(function* () {
    // use 'self' instead of 'this'
  });
}
```

**WASM Integration**: The GPAC WASM module exports C functions directly. TypeScript bindings wrap these with `ccall`/`cwrap`. Exported functions are defined in `build-wasm.sh` `-sEXPORTED_FUNCTIONS`.

**Coordinate System**: BIFS uses center origin with Y-up. Canvas2DRenderer transforms: `translate(width/2, height/2)` then `scale(1, -1)`.

## WASM Build Notes

- Requires Emscripten SDK (`source emsdk_env.sh` before building)
- SDL is disabled via post-configure patching of config.mak
- `src/wasm_stubs.c` provides stubs for functions removed with applications/
- Memory limit: 256MB WASM heap

## Security

CI runs CodeQL analysis and AddressSanitizer builds. Fuzzing targets BIFS decoder (see `.github/workflows/security.yml`).

## AOT Compiler


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bytebrujo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
