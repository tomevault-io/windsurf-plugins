---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Doxa is a **header-only C++ image binarization framework** that converts grayscale images to binary (black and white). The core library has zero external dependencies and is designed to be lightweight and easy to integrate with other frameworks (OpenCV, Qt, etc.). Language bindings exist for Python and WebAssembly/JavaScript.

## Build and Test Commands

The project uses CMake presets for unified builds. All commands run from the project root.

### Quick Start (CMake Presets)

```bash
# Build and run C++ unit tests
cmake --preset cpp-tests
cmake --build build-cpp-tests --config Release
ctest --test-dir build-cpp-tests -C Release

# Build and test Python bindings
cmake --preset python
cmake --build build-python --config Release
ctest --test-dir build-python -C Release

# Build and test WebAssembly (requires Emscripten in PATH)
cmake --preset wasm
cmake --build build-wasm --config Release
ctest --test-dir build-wasm -C Release

# Build and run performance benchmarks (Google Benchmark)
cmake --preset benchmarks
cmake --build build-bench --config Release
./build-bench/Doxa.Bench/doxa_bench              # Linux/Mac
.\build-bench\Doxa.Bench\Release\doxa_bench.exe  # Windows

# Build everything (C++, Python, WASM)
cmake --preset all
cmake --build build --config Release
ctest --test-dir build -C Release
```

**Note:** On Windows with Visual Studio (multi-config generator), `--config Release` and `-C Release` are required. On Linux/Mac with single-config generators (Make, Ninja), these flags are optional.

### C++ Unit Tests

The C++ test suite uses Google Test (fetched automatically via CMake).

```bash
# Using preset (recommended)
cmake --preset cpp-tests
cmake --build build --config Release
ctest --test-dir build -C Release

# Or build directly from Doxa.Test
cd Doxa.Test
cmake -S . -B ./build
cmake --build ./build --config Release
ctest --test-dir ./build -C Release  # Or run directly:
./build/doxa_test  # Linux/Mac
.\build\Release\doxa_test.exe  # Windows
```

### Python Bindings (DoxaPy)

DoxaPy requires Python 3.12+ and uses nanobind for C++ interop.

```bash
# Using preset (recommended)
cmake --preset python
cmake --build build-python --config Release
ctest --test-dir build-python -C Release

# Or build from Bindings/Python directory
cd Bindings/Python
pip install -r requirements.txt
cmake -S . -B ./build
cmake --build ./build --config Release
python test/test_doxa.py

# Build distributable wheel (uses cibuildwheel).
# Note: this is the only path that still requires copy-cpp-files.py
# (run it first), since sdist creation happens before CMake configures.
python copy-cpp-files.py
python -m build
```

### WebAssembly Bindings (DoxaJs)

DoxaJs uses CMake with Emscripten toolchain.

```bash
# Using preset (recommended, requires emcmake in PATH)
cmake --preset wasm
cmake --build build-wasm --config Release
ctest --test-dir build-wasm -C Release

# Or build directly with emcmake
emcmake cmake -S . -B build-wasm -DCMAKE_BUILD_TYPE=Release
cmake --build build-wasm --config Release
cd Bindings/WebAssembly && npm test

# Output: ./dist/doxaWasm.js and ./dist/doxaWasm.wasm
```

## Architecture

### Header-Only Core Library

The entire core library is in `Doxa/*.hpp` files. There is no build step for the core library - just include the headers.

**Key Files:**
- `Doxa/Image.hpp` - Core 8-bit image data structure with move semantics
- `Doxa/Algorithm.hpp` - Base algorithm interface using CRTP (Curiously Recurring Template Pattern)
- `Doxa/Parameters.hpp` - Key-value parameter system (supports `{"window": 75, "k": 0.2}`)
- `Doxa/PNM.hpp` - Image I/O for Portable Any-Map formats (PBM, PGM, PPM, PAM)
- `Doxa/BinarizationFactory.hpp` - Factory pattern for algorithm instantiation
- `Doxa/ClassifiedPerformance.hpp` - Performance metrics (Accuracy, F-Measure, Precision, Recall, MCC, PSNR, NRM)
- `Doxa/DRDM.hpp` - Distance-Reciprocal Distortion Measure metric

### Algorithm Organization

All binarization algorithms inherit from `Algorithm<T>` base class (CRTP pattern):

**Global Thresholding:**
- `Otsu.hpp` - Histogram-based single threshold

**Local Adaptive Thresholding:**
- `Sauvola.hpp`, `Niblack.hpp`, `Wolf.hpp`, `Nick.hpp`, `TRSingh.hpp`, `Bernsen.hpp`, `Phansalkar.hpp`, `ISauvola.hpp`, `Wan.hpp`, `Su.hpp`, `Gatos.hpp`, `Bataineh.hpp`, `AdOtsu.hpp`

Each algorithm can be called statically via:
```cpp
Image binaryImage = Sauvola::ToBinaryImage(grayImage, parameters);
```

Or instantiated for reuse:
```cpp
Sauvola sauvola;
sauvola.Initialize(grayImage);
sauvola.ToBinary(binaryImage, parameters);
```

### Calculation Optimizations

The framework includes multiple optimization strategies for local window operations:

- `ChanCalc.hpp` - Memory-efficient sliding window sums (Chan 2019)
- `IntegralImageMeanVarianceCalc.hpp` - Integral image acceleration (Shafait 2008)
- `LocalWindow.hpp` - Generic window iteration template
- `GridCalc.hpp` - Grid-based calculations

Algorithms use these via template inheritance to avoid virtual function overhead.

### Language Bindings Architecture

**Python (DoxaPy):**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brandonmpetty/Doxa](https://github.com/brandonmpetty/Doxa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
