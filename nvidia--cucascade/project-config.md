---
trigger: always_on
description: <!-- GSD:project-start source:PROJECT.md -->
---

<!-- GSD:project-start source:PROJECT.md -->
## Project

**cuCascade Disk I/O Performance Optimization**

Performance optimization of cuCascade's disk I/O backends (GDS and kvikIO) to approach raw hardware throughput on NVMe storage. The disk tier enables persisting GPU data batches to disk and reading them back, but current throughput is 5-11x below what the hardware can deliver. This project closes that gap.

**Core Value:** Both GDS and kvikIO disk I/O backends achieve within 80% of raw hardware throughput (gdsio/dd baselines) for read and write paths.

### Constraints

- **Buffer registration**: Cannot assume RMM-allocated GPU memory is pre-registered with cuFile — but CAN register temporarily per-transfer for large I/O
- **Thread safety**: Disk I/O operations must remain safe for concurrent use
- **RAII**: All file handles and disk resources must follow existing RAII patterns
- **API compatibility**: idisk_io_backend interface must not change (backends are swappable)
- **Benchmark path**: Benchmarks should use /mnt/disk_2 for NVMe-fair comparison with baselines
<!-- GSD:project-end -->

<!-- GSD:stack-start source:codebase/STACK.md -->
## Technology Stack

## Languages
- C++20 - All library source code (`src/memory/`, `src/data/`) and public headers (`include/cucascade/`)
- CUDA C++20 - GPU kernel code (`test/memory/test_gpu_kernels.cu`) and direct CUDA runtime calls throughout `src/`
- CMake 3.26.4+ - Build system (`CMakeLists.txt`, `cmake/`, `CMakePresets.json`)
- Python - Utility scripts (`scripts/generate_api_docs.py`, `scripts/compare_benchmarks.py`)
## Runtime
- Linux only: `linux-64` and `linux-aarch64` (enforced via `pixi.toml` line 4)
- NVIDIA GPU required; compute capability >= 7.5 (Turing or newer)
- CUDA Toolkit 12.9+ (cuda-12 track) or 13+ (cuda-13 track)
- NUMA-aware host: requires `libnuma` (installed via `numactl` pixi dependency)
- Pixi >= 0.59
- Config: `pixi.toml`
- Lockfile: `pixi.lock` (committed)
- Channels: `rapidsai-nightly`, `conda-forge` (default); `rapidsai`, `conda-forge` (cudf-stable feature)
## Frameworks
- RMM (RAPIDS Memory Manager) - GPU/host memory resource abstraction; provides `rmm::mr::device_memory_resource`, `rmm::cuda_stream_view`, `rmm::out_of_memory`, `rmm::bad_alloc`; pulled in via `find_package(rmm REQUIRED CONFIG)` from libcudf installation
- libcudf 26.06 (nightly) / 26.02 (stable) - Columnar data representation; provides `cudf::table`, `cudf::column`, `cudf::type_id`, `cudf::pack`/`unpack`; pulled in via `find_package(cudf REQUIRED CONFIG)`
- Catch2 v2.13.10 - Unit test framework; fetched via `FetchContent` in `test/CMakeLists.txt`; test executable: `cucascade_tests`
- Google Benchmark v1.8.3 - Microbenchmark framework; fetched via `FetchContent` in `benchmark/CMakeLists.txt`; benchmark executable: `cucascade_benchmarks`
- Ninja - Build generator (configured in `CMakePresets.json`)
- sccache - Compiler cache for C, CXX, and CUDA compilers (`CMAKE_C_COMPILER_LAUNCHER`, `CMAKE_CXX_COMPILER_LAUNCHER`, `CMAKE_CUDA_COMPILER_LAUNCHER` in `CMakePresets.json`)
- clang-format v20.1.4 - Code formatting enforced via pre-commit (`.clang-format` at repo root)
- cmake-format / cmake-lint v0.6.13 - CMake file linting via pre-commit
- black v25.1.0 - Python formatting via pre-commit
- codespell v2.4.1 - Spell checking via pre-commit (ignore list: `.codespell_words`)
- Doxygen - API documentation generation; config: `Doxyfile`; output parsed by `scripts/generate_api_docs.py`
## Key Dependencies
- `libcudf` 26.06 / 26.02 - Core data representation; `cudf::table` is the GPU-tier data container; all column type handling (LIST, STRUCT, STRING, DICTIONARY32, etc.) delegates to cudf
- `RMM` (via cudf) - `rmm::mr::device_memory_resource` is the base class for all custom allocators; `rmm::cuda_stream_view` is used throughout for CUDA stream propagation
- `CUDA::cudart` - Direct CUDA runtime API calls (`cudaMalloc`, `cudaMemcpyAsync`, `cudaStreamSynchronize`, `cudaFree`, `cudaMallocHost`, `cudaFreeHost`)
- `CUDA::nvml` - GPU topology discovery via NVML in `src/memory/topology_discovery.cpp`
- `kvikio` 26.06 / 26.02 - Async disk I/O with automatic GDS/POSIX fallback; used in `src/data/kvikio_io_backend.cpp` via `kvikio::FileHandle`; linked PRIVATE via `kvikio::kvikio`
- `libcufile` (cuFile / GDS) - NVIDIA GPUDirect Storage for direct GPU↔NVMe transfers; `<cufile.h>` used in `src/data/gds_io_backend.cpp`; found via `find_library(CUFILE_LIB cufile ...)` — optional at configure time, required at runtime for GDS backend
- `libnuma` - NUMA-aware pinned host memory allocation in `src/memory/numa_region_pinned_host_allocator.cpp`; found via `find_library(NUMA_LIB numa REQUIRED)`
- `Threads::Threads` (pthreads) - Thread support; `std::mutex`, `std::condition_variable`, `std::async` throughout
- `fmt` - Format library (pixi dependency; available in environment)
- `nvtx3::nvtx3-cpp` - NVIDIA NVTX profiling annotations; only linked when `CUCASCADE_NVTX=ON`; used via `CUCASCADE_FUNC_RANGE()` macro in `include/cucascade/error.hpp`
## Configuration
- `CUDAARCHS` - Set by pixi environment activation to select CUDA architecture targets
- `CMAKE_PREFIX_PATH` - Passed through from pixi environment for dependency resolution
- `CUCASCADE_BUILD_TESTS` (default ON) - Adds `test/` subdirectory

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA/cuCascade](https://github.com/NVIDIA/cuCascade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
