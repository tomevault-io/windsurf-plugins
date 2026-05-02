---
trigger: always_on
description: A minimal DuckDB WASM build for browsers and Cloudflare Workers.
---

# Ducklings

A minimal DuckDB WASM build for browsers and Cloudflare Workers.

## Goals

- Minimal footprint/size without sacrificing essential features
- TypeScript API for JavaScript/TypeScript access
- Remote data loading via httpfs extension
- Use Flechette instead of apache-arrow for JS/Arrow bridging
- First-class Cloudflare Workers support

## Packages

The project publishes two npm packages:

| Package | Description | API Style |
|---------|-------------|-----------|
| `@ducklings/browser` | Browser version | Sync |
| `@ducklings/workers` | Cloudflare Workers version | Async (Promises) |

**Why two packages?** Cloudflare Workers doesn't support synchronous XMLHttpRequest. The workers package uses Emscripten's Asyncify to enable async `fetch()` calls, making httpfs work properly.

## Project Structure

```
ducklings/
├── Makefile                       # Build orchestration
├── deps/                          # Git submodules
│   ├── duckdb/                    # DuckDB v1.5.0 (C++ source)
│   ├── duckdb-httpfs/             # httpfs extension source
│   ├── duckdb-iceberg/            # Iceberg extension source
│   └── duckdb-avro/               # Avro extension source (iceberg dependency)
├── dist/                          # WASM build output
│   ├── duckdb.js                  # Browser JS glue
│   ├── duckdb.wasm                # Browser WASM binary
│   ├── duckdb-workers.js          # Workers JS glue (Asyncify)
│   └── duckdb-workers.wasm        # Workers WASM binary
├── vcpkg.json                     # vcpkg manifest (iceberg/avro C deps)
├── vcpkg_ports/                   # vcpkg overlay ports for WASM compat
│   ├── aws-c-io/                  # No-op I/O for WASM
│   ├── liblzma/                   # LITTLE_ENDIAN fix for wasm32
│   ├── snappy/                    # fPIC fix for wasm32
│   └── zlib/                      # Build fixes
├── patches/                       # Source patches for dependencies
│   └── duckdb/                    # DuckDB-specific patches
│       └── preloaded_extensions.patch # Makes DuckDB recognize httpfs/avro/iceberg as preloaded
├── packages/
│   ├── ducklings-browser/         # @ducklings/browser (npm)
│   ├── ducklings-workers/         # @ducklings/workers (npm)
│   ├── example-browser/           # Browser example (private)
│   └── example-cloudflare-worker/ # CF Workers example (private)
├── scripts/
│   └── build-duckdb.sh            # Emscripten build script
└── .github/workflows/
    ├── ci.yml                     # CI (build, test on PR/push)
    └── release.yml                # Release (npm publish on tag)
```

## Build System

### Prerequisites

- Node.js 18+
- pnpm 9+
- Emscripten SDK 4.0.22
- Binaryen (for wasm-opt)
- git (for vcpkg clone during build)

### Make Targets

```bash
# Full build
make all                  # Build browser WASM + TypeScript package

# WASM compilation (requires Emscripten)
make duckdb-browser       # Compile browser WASM (~2 min)
make duckdb-workers       # Compile workers WASM with Asyncify (~3 min)
make duckdb-all           # Build both WASM variants

# TypeScript packages
make typescript-browser   # Build @ducklings/browser package
make typescript-workers   # Build @ducklings/workers package
make typescript-all       # Build both packages

# Utilities
make deps                 # Initialize git submodules
make clean                # Clean build artifacts
```

### Build Flow

1. `make duckdb-browser` compiles DuckDB C++ to WASM using Emscripten
2. Output goes to `dist/duckdb.js` and `dist/duckdb.wasm`
3. `make typescript-browser` builds the npm package
4. Package build copies WASM files to `packages/ducklings-browser/dist/wasm/`

Same flow for workers variant with Asyncify enabled.

## Key Files

### TypeScript API

- `packages/ducklings-browser/src/index.ts` - Browser API (sync)
- `packages/ducklings-workers/src/index.ts` - Workers API (async, standalone)

Both packages export:
- `init()` - Initialize WASM module
- `DuckDB` - Database class
- `Connection` - Connection with query/execute methods
- `PreparedStatement` - Parameterized queries
- `version()` - Get DuckDB version
- Flechette re-exports: `tableFromArrays`, `tableFromIPC`, `tableToIPC`

### WASM Build

- `scripts/build-duckdb.sh` - Main build script
- Uses Emscripten with -Oz optimization, LTO, wasm-opt
- Extensions built-in: Parquet, httpfs, JSON, Avro, Iceberg
- Workers build adds `-sASYNCIFY` for async fetch support
- vcpkg provides C/C++ dependencies (AWS SDK, roaring, avro-c, curl, openssl, etc.)

### Patch System

DuckDB source requires patches for WASM compatibility. Patches are stored in `patches/duckdb/` and applied automatically during build.

**Current Patches:**

| Patch | Purpose |
|-------|---------|
| `preloaded_extensions.patch` | Makes DuckDB recognize httpfs, avro, and iceberg as preloaded |
| `http_util_extension_guard.patch` | Guards HTTP client code behind DUCKDB_DISABLE_EXTENSION_LOAD |

**How it works:**
1. Build script calls `apply_patches()` before CMake configuration
2. Patches are applied with `patch -p1` from the DuckDB source directory
3. A marker check prevents re-applying already-applied patches
4. `make clean` resets the submodule to remove patches

**Updating DuckDB version:**
1. Update `DUCKDB_VERSION` in Makefile

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tobilg/ducklings](https://github.com/tobilg/ducklings) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
