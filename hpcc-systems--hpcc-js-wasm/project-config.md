---
trigger: always_on
description: **ALWAYS follow these instructions first**. Only fallback to additional search and context gathering if the information here is incomplete or found to be in error.
---

# HPCC-JS-WASM Copilot Instructions

**ALWAYS follow these instructions first**. Only fallback to additional search and context gathering if the information here is incomplete or found to be in error.

## Repository Overview

This is a WebAssembly (WASM) monorepo providing JavaScript/TypeScript bindings for C++ libraries:
- @hpcc-js/wasm-base91 - Base91 encoding/decoding
- @hpcc-js/wasm-duckdb - DuckDB embedded database
- @hpcc-js/wasm-expat - Expat XML parser
- @hpcc-js/wasm-graphviz - Graphviz graph visualization
- @hpcc-js/wasm-graphviz-cli - Graphviz CLI tool
- @hpcc-js/wasm-llama - Llama.cpp AI models
- @hpcc-js/wasm-zstd - Zstandard compression
- @hpcc-js/wasm - Meta package for backward compatibility

## Critical Build Requirements

**NEVER CANCEL BUILD COMMANDS** - Full builds can take 45+ minutes. ALWAYS set timeouts to 60+ minutes for any build command.

### Initial Setup Commands
```bash
# 1. Install dependencies (2-3 minutes)
npm ci

# 2. Install build dependencies (20-30 minutes - NEVER CANCEL)
# Includes: emsdk install (~5 min), vcpkg install (~15-20 min), playwright (~5-10 min), bundler test deps
npm run install-build-deps

# 3. Build C++ to WASM (30-45 minutes - NEVER CANCEL) 
# CRITICAL: Set timeout to 3600+ seconds (60+ minutes)
npm run build-cpp

# 4. Build TypeScript packages (5-10 minutes)
npm run build-ws

# 5. Full build command equivalent to build-cpp + build-ws (45+ minutes total - NEVER CANCEL)
# Equivalent to: build-cpp && build-ws
npm run build
```

### Measured Timing Examples
From actual runs in clean environment:
- `npm ci`: 44 seconds
- `npm run lint`: 1-2 minutes (after TypeScript build)
- `npm run install-emsdk`: 5-10 minutes (downloading + setup)
- `npm run install-vcpkg`: 15-25 minutes (depends on network)
- `npx playwright install`: 10-20 minutes (browser downloads)
- `npm run build-cpp`: 30-60 minutes (C++ compilation to WASM)
- `npm run test`: 15-30 minutes (includes browser and node tests)

## CMake / C++ (WASM) Build

Prefer the npm scripts. They source `./emsdk/emsdk_env.sh` and use the repo's CMake preset(s):

```bash
npm run build-cpp
```

For fast iteration:

```bash
npm run build-cpp-watch
```

### Quick Development (TypeScript Only)
```bash
# For TypeScript-only changes when WASM files exist
npm ci                  # 2-3 minutes
npm run build-ws        # 5-10 minutes
npm run lint           # 2-3 minutes
```

### Manual CMake (only when needed)

Manual CMake is supported, but you must source Emscripten first:

```bash
source ./emsdk/emsdk_env.sh
cmake -S . -B ./build --preset vcpkg-emscripten-MinSizeRel
cmake --build ./build --parallel
```

## Embind (C++ <-> JS Interop)

This repo uses Emscripten Embind to expose C++ APIs to JavaScript/TypeScript.

Reference: https://emscripten.org/docs/porting/connecting_cpp_and_javascript/embind.html

### Guidelines
- Prefer Embind over ad-hoc JS glue when exposing C++ to JS.
- Avoid throwing across the WASM boundary; catch exceptions in C++ and translate into explicit error results.
- Be explicit about ownership/lifetime:
  - If you return raw pointers (often created with `new`) via Embind, JS must eventually free them (e.g. via the generated `.delete()` method or `Module.destroy(obj)` depending on how the wrapper is used).
  - If practical, bind smart pointers (e.g. `std::unique_ptr` / `std::shared_ptr`) to make ownership explicit.
- For JS values, prefer `emscripten::val` and convert at the boundary (e.g. arrays -> `val.isArray()` + `val["length"]`). Validate types and reject non-finite numbers.
- Keep interop-friendly result types: materialize streaming results where needed for JS access patterns.

### Common Patterns Used Here
- `EMSCRIPTEN_BINDINGS(...)` with `emscripten::class_<T>` and `.function(...)`.
- `allow_raw_pointers()` is used for functions that accept/return pointer types.
- Provide small helper conversions (JS `val` -> C++ types) and centralize error messages for predictable JS behavior.

## Working Without Full Build

### What Works with npm ci + npm run lint Only
- Code style checking and linting
- TypeScript type checking (tsc --noEmit)
- Documentation viewing and editing
- Package.json script modification
- Configuration file changes

### What Requires npm run build-ws
- TypeScript compilation to JavaScript
- Package distribution files (dist/ directories)
- Import/export resolution testing
- ESLint with compiled output

### What Requires Full WASM Build (npm run build-cpp + build-ws)
- Running any package functionality
- All tests (browser and node)
- Bundle testing (webpack, rollup, esbuild)
- Actual library usage and validation
- Performance testing

**WARNING**: TypeScript-only builds WILL FAIL if WASM files don't exist. You'll see errors like:
```
✘ [ERROR] Could not resolve "../../../build/packages/base91/src-cpp/base91lib.wasm"
✘ [ERROR] Could not resolve "../../../build/packages/graphviz/src-cpp/graphvizlib.wasm"
```

## Testing

### Prerequisites
```bash
# Install playwright browsers (10-15 minutes - NEVER CANCEL)
npx playwright install --with-deps
```

### Test Commands
```bash
# Run all tests (15-20 minutes - NEVER CANCEL)
npm run test

# Run specific package tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hpcc-systems/hpcc-js-wasm](https://github.com/hpcc-systems/hpcc-js-wasm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
