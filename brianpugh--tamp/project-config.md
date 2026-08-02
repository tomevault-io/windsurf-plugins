---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

## Project Overview

Tamp is a low-memory, DEFLATE-inspired lossless compression library designed for
embedded targets. The project contains multiple implementations targeting
different platforms:

### Core Architecture

**Multi-language Implementation Strategy:**

- **C Library** (`tamp/_c_src/tamp/`) - Core implementation with
  compressor/decompressor
- **Python Bindings** - Multiple implementations:
  - Pure Python reference (`tamp/compressor.py`, `tamp/decompressor.py`)
  - Cython-accelerated C bindings (`tamp/_c_*.pyx`) - primary distribution
  - MicroPython Viper (`tamp/*_viper.py`)
  - MicroPython Native Module (`mpy_bindings/`)
- **WebAssembly** (`wasm/`) - JavaScript/TypeScript bindings via Emscripten
- **ESP-IDF Component** (`espidf/`) - ESP32 optimized version

**Shared C Source:** All implementations use the same C source code in
`tamp/_c_src/tamp/`:

- `common.h/c` - Shared utilities, data structures, stream I/O callbacks, and
  dictionary initialization
- `compressor.h/c` - Compression implementation (sink/poll low-level API and
  higher-level compress/flush API)
- `decompressor.h/c` - Decompression implementation
- `compressor_find_match_desktop.c` - Desktop-optimized match finding (included
  by `compressor.c` on 64-bit targets: x86_64, aarch64, unless
  `TAMP_USE_EMBEDDED_MATCH=1`)

## Development Commands

### Python Development

**Environment Setup:**

```bash
uv sync                    # Install dependencies (also builds the Cython extensions)
source .venv/bin/activate  # Activate virtual environment
```

**Build and Test:**

```bash
# Build/rebuild Cython extensions (uv sync recompiles when C/Cython sources change)
uv sync

# Run tests
uv run pytest                    # All tests
uv run pytest tests/test_compressor.py  # Specific test file

# Run both Python and MicroPython tests
make test

# CLI usage
uv run tamp compress input.txt -o output.tamp
uv run tamp decompress output.tamp -o restored.txt
```

**Code Quality:**

```bash
uv run ruff check              # Linting
uv run ruff format             # Formatting
uv run pyright                 # Type checking
```

**Testing with AddressSanitizer (Linux only):**

```bash
# Build with sanitizers enabled (--reinstall-package forces a rebuild since the
# env var changed but the sources did not)
TAMP_SANITIZE=1 uv sync --reinstall-package tamp

# Run tests with AddressSanitizer (requires LD_PRELOAD on Linux)
LD_PRELOAD=$(gcc -print-file-name=libasan.so) \
ASAN_OPTIONS=detect_leaks=0 \
UBSAN_OPTIONS=print_stacktrace=1 \
uv run pytest

# Note: AddressSanitizer is only supported on Linux due to security
# restrictions on macOS that prevent LD_PRELOAD/DYLD_INSERT_LIBRARIES
```

### WebAssembly Development

**Prerequisites:** Emscripten SDK must be installed and activated

```bash
cd wasm/
activate emsdk              # Activate Emscripten environment
```

**Build and Test:**

```bash
npm run build              # Production build (make all && npm run build:js)
npm run build:debug       # Debug build with symbols
npm test                   # All tests using Node.js test runner
npm run test:basic         # Core functionality tests
npm run test:integration   # API integration tests
```

**Code Quality:**

```bash
npm run format-and-lint    # Prettier + ESLint
npm run type-check         # TypeScript checking
```

### MicroPython Development

**Native Module Build:**

```bash
# Requires MPY_DIR environment variable pointing to MicroPython source
make ARCH=armv6m           # Build for specific architecture
```

Changing `ARCH` alone does not trigger a rebuild (stale objects are reused; make
reports "Nothing to be done"). Remove `build/mpy_bindings/` and `build/tamp/`
when switching architectures.

**Testing on Device:**

```bash
make on-device-compression-benchmark    # Requires BELAY_DEVICE env var
make on-device-decompression-benchmark
```

### C Library Development

**Build Static Library:**

```bash
make tamp-c-library        # Creates build/tamp.a
```

**Run C Tests:**

```bash
make c-test               # Unit tests using Unity framework
```

### On-Device Testing (`devices/`)

Per-device benchmark/test harnesses; results tables live in
`devices/BENCHMARKS.md` (standard workload: first 100 KB of enwik8, 10-bit
window). The harness logic (benchmarks, reference verification,
regression-vector replay, PRNG stress, `TAMP-DEVICE-RESULT:` sentinel) is shared
in `devices/common/tamp_bench.c`; each device provides a thin `main` plus
`tamp_bench_time_us()`. Regression vectors are shared in `devices/vectors/`.

`devices/espidf/` — ESP-IDF harness that benchmarks and correctness-tests tamp
on real ESP32 hardware using the local `espidf/tamp/` component. Requires an
activated esp-idf environment.

```bash
make esp32-device-build                          # Build (ESP32_TARGET=esp32s3 default)
make esp32-device-test ESP32_PORT=/dev/ttyUSB0   # Build, flash, run; nonzero exit on failure
make esp32-device-benchmark ESP32_PORT=...      # Same, plus BENCH summary
```

`TAMP_ESP32_OPT=n` builds the `TAMP_ESP32=n` variant into a separate build dir
for A/B comparison. Regression vectors in `devices/vectors/` (e.g. host-fuzzer

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BrianPugh/tamp](https://github.com/BrianPugh/tamp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
