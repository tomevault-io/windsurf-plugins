---
trigger: always_on
description: **FusedKernelLibrary (FKL)** is a C++20 header-only library that enables automatic GPU kernel fusion without requiring CUDA expertise. It implements four fusion techniques:
---

# Copilot Instructions for FusedKernelLibrary

## Project Overview

**FusedKernelLibrary (FKL)** is a C++20 header-only library that enables automatic GPU kernel fusion without requiring CUDA expertise. It implements four fusion techniques:
- **Vertical Fusion**: Chain operations into a single kernel with no intermediate memory writes.
- **Horizontal Fusion**: Process multiple data planes in parallel using `blockIdx.z`.
- **Backwards Vertical Fusion**: Read-backwards through a pipeline (like OpenCV Filters), computing only required pixels.
- **Divergent Horizontal Fusion**: Execute different kernels simultaneously using different SM components.

The library has CPU and CUDA backends. HIP support is architecturally possible but not yet implemented.

**License**: Apache 2.0  
**Version**: 0.2.0 (main branch — API may break for maintainability)  
**LTS branch**: `LTS-C++17` (frozen API, C++17 minimum, adds features cautiously)

---

## Repository Layout

```
FusedKernelLibrary/
├── .clang-format               # LLVM-based style, 4-space indent, 120-char column limit
├── .github/workflows/          # CI: cmake-linux-amd64.yml, cmake-linux-arm64.yml, cmake-windows-amd64.yml
├── CMakeLists.txt              # Root build (v0.2.0, requires CMake >= 3.24, C++ and optional CUDA)
├── cmake/                      # CMake helpers: arch flags, CUDA init, test discovery, generators
│   ├── archflags.cmake         # CPU SIMD flags (AVX2 default on MSVC x64, native on Unix)
│   ├── cmake_init.cmake        # Global CMake settings
│   ├── cuda_init.cmake         # CUDA language enablement and NVCC path (Ninja/Windows workaround)
│   ├── libs/cuda/archs.cmake   # CUDA arch selection/filtering (requires compute_70+ for CUDA < 13)
│   └── tests/                  # Test discovery and stub generation
│       ├── discover_tests.cmake
│       └── add_generated_test.cmake
├── include/fused_kernel/       # All public headers (header-only library)
│   ├── fused_kernel.h          # Main API entry point (includes executors.h)
│   ├── algorithms/             # Operations: arithmetic, cast, image processing, etc.
│   └── core/                   # Infrastructure: execution model, data types, utils
│       ├── execution_model/    # Executors, DPP patterns, operation model
│       ├── data/               # Ptr, Ptr2D, Tensor, RawPtr types
│       └── utils/              # Macros (utils.h), compiler detection (compiler_macros.h)
├── lib/                        # CMake library target definition and version config
├── tests/                      # Integration tests (header .h files, auto-discovered)
├── utests/                     # Unit tests (header .h files, auto-discovered)
└── benchmarks/                 # Benchmarks (disabled by default, ENABLE_BENCHMARK=ON)
```

---

## Build System

### Requirements
- **CMake** >= 3.24
- **C++ compiler** with C++20 support
- **CUDA** (optional): requires NVCC. **Only nvcc is supported as the CUDA compiler**; clang-as-CUDA-compiler is not supported despite `CLANG_HOST_DEVICE` macro existing.
- **MSVC**: Visual Studio 2019+ (MSVC_VERSION >= 1920) required; older versions disable the CPU backend.

### Configure and Build (typical)
```bash
# Linux (Ninja)
cmake -G "Ninja" -B build -DCMAKE_BUILD_TYPE=Release -S .
cmake --build build --config Release

# Windows (Ninja, inside VS Developer Shell)
cmake -G "Ninja" -B build -DCMAKE_BUILD_TYPE=Release -S .
cmake --build build --config Release
```

### Key CMake Options
| Option | Default | Description |
|--------|---------|-------------|
| `ENABLE_CPU` | `ON` | Build CPU backend (auto-disabled for MSVC < 2019) |
| `ENABLE_CUDA` | `ON` (if NVCC found) | Build CUDA backend |
| `BUILD_TEST` | `ON` | Build integration tests |
| `BUILD_UTEST` | `ON` | Build unit tests |
| `ENABLE_BENCHMARK` | `OFF` | Build benchmark targets |
| `CUDA_ARCH` | `"native"` | CUDA architecture(s); use `"native"`, `"all"`, `"all-major"`, or explicit list |
| `ARCH_FLAGS` | `AVX2`/`native` | CPU SIMD flags (MSVC: AVX/AVX2/AVX512; Unix: native/haswell/…) |

### CUDA Architecture Notes
- **CUDA < 13**: Architectures below `compute_70` (Volta) are filtered out automatically. A GPU with compute < 70 will trigger an error.
- **`native` with CUDA < 13**: `nvidia-smi --query-gpu=compute_cap` is executed at CMake configure time to detect the local GPU.
- **CUDA >= 13**: All architectures allowed.

### Windows-Specific Notes
- CI uses self-hosted runners with LLVM 21.1.0 at `D:/clang+llvm-21.1.0-x86_64-pc-windows-msvc/bin/`.
- The Ninja generator requires a workaround: after `cmake`, `rules.ninja` may contain a wrong NVCC path that must be patched (see `cmake-windows-amd64.yml`).
- The VS Developer Shell (`Enter-VsDevShell`) must be activated for both configure and build steps on Windows.
- `utf8cp.manifest` is embedded into test executables on Windows for UTF-8 codepage support.

---

## Running Tests

```bash
cd build
ctest --build-config Release --output-junit test_results.xml
```

Tests are registered with CTest automatically. Individual targets follow the naming pattern `<TestName>_cpp` (CPU) and `<TestName>_cu` (CUDA).

---

## Test Infrastructure

### How Tests Are Discovered

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Libraries-Openly-Fused/FusedKernelLibrary](https://github.com/Libraries-Openly-Fused/FusedKernelLibrary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
