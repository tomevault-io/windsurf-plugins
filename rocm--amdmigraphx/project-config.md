---
trigger: always_on
description: This file provides guidance for agents and contributors working with code in this repository.
---

# Repository Guidelines

This file provides guidance for agents and contributors working with code in this repository.

## Project Overview

MIGraphX is AMD's graph inference engine that accelerates machine learning model inference on ROCm. It uses a three-level IR hierarchy:
- **Programs** (`migraphx::program`): Top-level containers for entire neural networks
- **Modules** (`migraphx::module`): DAGs of instructions representing computational subgraphs
- **Instructions** (`migraphx::instruction`): Atomic operations wrapping an `operation` with inputs

The compilation flow transforms high-level models (ONNX, TensorFlow) through optimization passes into target-specific code (GPU kernels via HIP/MIOpen/rocBLAS, CPU ops, or reference implementations).

## Dependencies

**ROCm ecosystem:** ROCm, MIOpen, rocBLAS, hipBLASLt, HIP (paths under `/opt/rocm`)
**Core libraries:** Protobuf (ONNX), Half (IEEE 754 fp16), pybind11 (Python), nlohmann/json, MessagePack, SQLite3
**Build tools:** CMake 3.15+, rbuild (optional but recommended), ccache (via rbuild develop)
**Language level:** C++17 (set in CMake)

## Project Structure & Module Organization

- `src/include/migraphx/*.hpp` - Public API headers
- `src/op/` - Operation implementations
- `src/targets/gpu/` - GPU backend (HIP, MIOpen, rocBLAS lowering)
- `src/targets/cpu/` - CPU backend
- `src/targets/ref/` - Reference implementation
- `src/driver/` - CLI tools (main.cpp, perf.cpp, verify.cpp, passes.cpp)
- `src/py/` - Python bindings (pybind11)
- `test/` - Unit and integration tests (mirrors `src/` structure)
- `test/gpu/` - GPU-specific tests (build as `test_gpu_<topic>`)
- `test/verify/` - End-to-end verification tests
- `test/onnx/` - ONNX model parsing/verification
- `tools/` and `cmake/` - Build helpers, scripts, format scripts, CMake modules
- `examples/` - Usage examples (C++ API, Python, diffusion, transformers, vision)
- `docs/` - Sphinx/Doxygen documentation; build artifacts in `docs/_build/`
- `build/` - Default generated build output

## Build Commands

**Development build (recommended):**
```bash
rbuild develop -DGPU_TARGETS=$(/opt/rocm/bin/rocminfo | grep -o -m1 'gfx.*')
```

**Manual CMake build:**
```bash
cmake -S . -B build -DGPU_TARGETS=$(/opt/rocm/bin/rocminfo | grep -o -m1 'gfx.*')
make -C build -j$(nproc)
```

**Build with dependencies (rbuild):**
```bash
rbuild build -d depend -B build -DGPU_TARGETS=$(/opt/rocm/bin/rocminfo | grep -o -m1 'gfx.*')
```

**Run all tests:**
```bash
make -C build check
```

**Install:**
```bash
make -C build install
```

**Build documentation:**
```bash
make -C build doc
# Or manually in docs/:
pip3 install -r sphinx/requirements.txt
python3 -m sphinx -T -E -b html -d _build/doctrees -D language=en . _build/html
```

**Format code:**
```bash
python3 tools/format.py -i
```

**Install git hooks:**
```bash
./.githooks/install
```

## Coding Standards

### Naming Conventions

- **Files and symbols**: `snake_case` (e.g., `compute_shape`, `eliminate_concat.cpp`)
- **Template parameters**: `CamelCase`
- **Macros**: `UPPER_CASE` with `MIGRAPHX_` prefix
- Public headers go under `src/include/migraphx/*.hpp`
- Prefer clear names over abbreviations
- Use `using type_alias = int` not `typedef int type_alias`
- Use `nullptr` not `NULL` or `0`
- Use `or` and `and` instead of `&&` and `||`

### Formatting (.clang-format)

- 4-space indent, no tabs
- 100 column limit
- Braces on new line for classes/functions/control structures
- Align consecutive assignments

### Linting

- `.clang-tidy` runs via CMake; treat warnings seriously (they are enforced in CI)
- `cppcheck` is enabled with custom rules

### Style Guidelines

- **Avoid raw loops** - Prefer algorithms from `<migraphx/algorithm.hpp>` and STL `<algorithm>`:
  ```cpp
  // Good: Declarative, expresses intent clearly
  std::transform(in.begin(), in.end(), std::back_inserter(out),
                 [](int i) { return i * 2; });

  // Avoid: Raw loops are error-prone and less optimizable
  for (int i : in) { out.push_back(i * 2); }

  // Note: std::for_each should NOT be used as it doesn't encapsulate a raw loop
  // It's just a wrapper around a for loop - use std::transform or other algorithms instead
  ```

  **MIGraphX-specific algorithms** (`#include <migraphx/algorithm.hpp>`):
  - `transform_if(start, last, out, pred, f)` - Transform with filtering
  - `transform_accumulate(first, last, init, binop, unaryop)` - Accumulate with projection
  - `transform_partial_sum(first, last, out, binop, unaryop)` - Partial sum with projection
  - `group_by(start, last, out, pred)` - Group elements by predicate
  - `group_unique(start, last, out, pred)` - Group consecutive unique elements
  - `group_find(start, last, pred, out)` - Find and group matching ranges
  - `adjacent_remove_if(first, last, pred)` - Remove based on adjacent pairs
  - `adjacent_for_each(first, last, f)` - Iterate over adjacent pairs
  - `for_each(first1, last1, first2, f)` - Two-range iteration (like std::transform)

  **If no suitable algorithm exists:** Add a new algorithm to `migraphx/algorithm.hpp` rather than using raw loops

- **Memory management** - Use `std::make_unique/shared`, avoid raw `new`/`delete`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ROCm/AMDMIGraphX](https://github.com/ROCm/AMDMIGraphX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
