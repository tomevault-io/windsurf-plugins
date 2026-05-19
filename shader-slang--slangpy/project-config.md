---
trigger: always_on
description: This file provides guidance to GitHub Copilot when working with code in this repository.
---

# Copilot Instructions for SlangPy

This file provides guidance to GitHub Copilot when working with code in this repository.

## Project Overview

SlangPy is a native Python extension that provides a high-level interface for working with low-level graphics APIs (Vulkan, Direct3D 12, CUDA). The native side wraps the slang-rhi project (`external/slang-rhi`) using nanobind bindings. The project also contains a "functional API" that allows users to call Slang functions on the GPU with Python function call syntax.

## Directory Structure

| Directory | Description |
|-----------|-------------|
| `src/sgl/` | Native C++ code (core GPU abstraction layer) |
| `src/slangpy_ext/` | Python bindings (nanobind) |
| `src/slangpy_torch/` | Native torch integration extension |
| `slangpy/` | Python package implementation |
| `slangpy/tests/` | Python tests (pytest) |
| `tests/` | C++ tests (doctest) |
| `tools/` | General utility scripts |
| `.github/workflows/` | CI workflows |
| `examples/`, `samples/examples/`, `samples/experiments/` | Example code |
| `docs/` | Documentation |
| `external/` | External C++ dependencies |

## Communication Style
- If I tell you that you are wrong, think about whether or not you think that's true and respond with facts.
- Avoid apologizing or making conciliatory statements.
- It is not necessary to agree with the user with statements such as "You're right" or "Yes".
- Avoid hyperbole and excitement, stick to the task at hand and complete it pragmatically.

## Key Rules

1. **New Python APIs must have tests** in `slangpy/tests/`
2. **Always build before running tests**
3. **Run pre-commit after completing tasks** (`pre-commit run --all-files`; re-run if it modifies files)
4. **Use type annotations** for all Python function arguments
5. **Minimize new dependencies** — the project has minimal external deps

## Architecture

The project has three main layers:
1. **Python Layer** (`slangpy/`) — High-level API with Module, Function, Device classes
2. **C++ Binding Layer** (`src/slangpy_ext/`) — Nanobind-based Python-C++ interface
3. **Core SGL Layer** (`src/sgl/`) — Low-level GPU device management and shader compilation

C++ types typically map to slang-rhi counterparts (e.g., `Device` wraps `rhi::IDevice`).

### Key Components

- **Module**: Container for Slang shader code, loaded from `.slang` files
- **Function**: Callable GPU function with automatic Python↔GPU marshalling
- **Device**: GPU context managing resources and compute dispatch
- **CallData**: Cached execution plans for optimized repeated calls
- **Buffer/Texture**: GPU memory resources with Python array interface

### Error Handling

- Python-layer errors raise standard Python exceptions (typically `ValueError`, `TypeError`, or `SlangPyError`).
- C++ errors are translated to Python exceptions via nanobind. Shader compile errors surface as exceptions containing the Slang compiler diagnostic text.
- GPU errors (device lost, out of memory) propagate as exceptions from the RHI layer.
- When debugging, set `SLANGPY_PRINT_GENERATED_SHADERS=1` to see the generated kernel code that gets compiled.

## Build Commands

Replace `<PLATFORM>` with `windows-msvc`, `linux-gcc`, or `macos-arm64-clang` as appropriate:

```bash
cmake --preset <PLATFORM>                    # Configure
cmake --build --preset <PLATFORM>-debug      # Build (debug)
cmake --preset <PLATFORM> --fresh            # Reconfigure from scratch
```

Available presets: `windows-msvc`, `windows-arm64-msvc`, `linux-gcc`, `macos-arm64-clang`.

## Testing

**Always build before running tests.**

```bash
pytest slangpy/tests -v                        # All Python tests
pytest samples/tests -vra                      # Example tests
python tools/ci.py unit-test-cpp               # C++ unit tests
pytest slangpy/tests/slangpy_tests/test_X.py -v          # Specific file
pytest slangpy/tests/slangpy_tests/test_X.py::test_fn -v # Specific function
```

Debug generated shaders (PowerShell):
```bash
$env:SLANGPY_PRINT_GENERATED_SHADERS="1"; pytest slangpy/tests/slangpy_tests/test_X.py -v
```

## Code Style

### C++
- **Classes**: PascalCase | **Functions/variables**: snake_case | **Members**: `m_` prefix

### Python
- **Classes**: PascalCase | **Functions/variables**: snake_case | **Public Members**: no prefix | **Private Members**: `_` prefix
- **All arguments must have type annotations**

## Documentation Style

### C++ (Doxygen)

```cpp
/// Description.
void do_something();

/// Pack two float values to 8-bit snorm.
/// @param v Float values in [-1,1].
/// @param options Packing options.
/// @return 8-bit snorm values in low bits, high bits all zero.
uint32_t pack_snorm2x8(float2 v, const PackOptions options = PackOptions::safe);
```

### Python (Sphinx)

```python
def myfunc(x: int, y: int) -> int:
    """
    Description.

    :param x: Some parameter.
    :param y: Some parameter.
    :return: Some return value.
    """
```

## Slang Language Basics

Slang is a shader language based on HLSL. Key patterns used in this project:
- `[shader("compute")]` attribute marks GPU entry points
- `StructuredBuffer<T>` / `RWStructuredBuffer<T>` for typed GPU arrays
- `uint3 tid : SV_DispatchThreadID` for thread indexing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shader-slang/slangpy](https://github.com/shader-slang/slangpy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
