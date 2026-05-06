---
trigger: always_on
description: High-performance header-only container library for C++23 providing:
---

# Fast Containers - C++ SIMD Containers

## Overview

High-performance header-only container library for C++23 providing:
- **btree**: Cache-friendly B+tree with SIMD search and hugepage support
- **dense_map**: Fixed-size sorted array with SIMD operations
- **Hugepage allocators**: Pooling allocators for TLB optimization

## Stack
- C++23, CMake 3.30+, Catch2 v3.11.0
- Style: Google C++ (clang-format), 80 chars, 2 spaces, `int* ptr`

## Repository Structure

```
include/fast_containers/        # Public API headers
  dense_map.hpp, dense_map.ipp, dense_map_simd.ipp
  btree.hpp, btree.ipp
  hugepage_allocator.hpp, policy_based_hugepage_allocator.hpp, hugepage_pool.hpp
tests/                          # Unit tests (Catch2)
  test_dense_map.cpp, test_btree.cpp, test_hugepage_allocator.cpp, test_policy_based_allocator.cpp
src/
  benchmarks/                   # Google Benchmark microbenchmarks
    dense_map_search_benchmark.cpp, hugepage_allocator_benchmark.cpp
  binary/                       # btree_benchmark, btree_stress
scripts/
  interleaved_btree_benchmark.py  # A/B testing harness for rigorous benchmarking
third_party/                    # Git submodules (catch2, benchmark, histograms, abseil-cpp, lyra, unordered_dense)
```

**Header-Implementation Separation**: Template implementations are in `.ipp` files (included at end of `.hpp`) for cleaner interfaces.

## Build Commands

| Build Type | Command | Key Flags |
|------------|---------|-----------|
| Debug | `cmake -S . -B cmake-build-debug && cmake --build cmake-build-debug --parallel` | None |
| Debug+AVX2 | `cmake -S . -B cmake-build-debug -DENABLE_AVX2=ON && cmake --build cmake-build-debug --parallel` | -mavx2 -mfma |
| Debug+ASAN | `cmake -S . -B cmake-build-asan -DENABLE_ASAN=ON && cmake --build cmake-build-asan --parallel` | -fsanitize=address |
| Release | `cmake -S . -B cmake-build-release -DCMAKE_BUILD_TYPE=Release && cmake --build cmake-build-release --parallel` | -O3 -mavx2 -march=haswell |

**Quick commands**:
- Test: `ctest --test-dir cmake-build-{debug,release,asan} --output-on-failure`
- Format: `cmake --build cmake-build-debug --target format`
- Clang-tidy: `cmake --build build --target clang-tidy`

**CMake Presets**: Use `cmake --list-presets` to see available presets, then:
```bash
cmake --preset release && cmake --build --preset release && ctest --preset release
```

## dense_map<Key, Value, Length, Compare, SearchMode>

### Core Design
- **Storage**: Separate `std::array<Key, Length>` and `std::array<Value, Length>` (SoA layout)
- **Rationale**: Enable SIMD key scans, cache-friendly searches
- **Constraint**: Fixed compile-time size, keys always const
- **Complexity**: Find O(log n), Insert/Remove O(n)

### Iterator Limitations (proxy pattern)
```cpp
// ❌ for (auto& pair : arr)      // Won't compile
// ✅ for (auto pair : arr)       // Correct - proxy still modifies array
// ❌ it->first = val;            // first is const
// ✅ it->second = val;           // second is mutable
```

## Performance: SearchMode Selection

| SearchMode | Use When | Size | Workload |
|------------|----------|------|----------|
| **SIMD** | Read-heavy | >32 | Negative lookups, find-dominant |
| **Linear** | Write-heavy | <32 | Frequent insert/erase, early exit benefits |
| **Binary** | Balanced | >64 | Mixed read/write, or no AVX2 |

### SIMD Find Performance (vs Linear)
- Size 32: 45% faster
- Size 64: 59% faster

### Why SIMD Loses on Small Arrays
**Root Cause**: IPC bottleneck, not cache misses
- Linear: IPC 5.76, 26.0% cache miss
- SIMD: IPC 3.68, 28.5% cache miss
- Simple scalar ops pipeline better than complex SIMD on small data

### SIMD with Large Values (>128 bytes)
SIMD provides minimal benefit (~1%) when performance is dominated by:
- Value movement during splits/merges
- Cache misses from large values
- Memory bandwidth, not comparison speed

**Best use cases for SIMD**:
- Small values (<64 bytes)
- Read-heavy workloads
- Smaller node sizes (more comparisons per operation)

## SIMD Implementation Details

- AVX2: `_mm256_cmpeq_epi32`, `_mm256_movemask_epi8`
- Scans 8 int32 keys in parallel
- Progressive chunking for data movement: AVX2 (32B) → SSE (16B) → scalar (1B)
- Requires: `std::is_trivially_copyable_v<T>`
- Cache line alignment: `alignas(64)` for 4.3% cache improvement
- **Supports std::greater**: Zero-overhead compile-time dispatch for ascending/descending order

## Byte Array SIMD: Not Viable

**Attempted**: SIMD support for arbitrary byte arrays (16/32-byte keys)

**Result**: 2-3× slower than binary search due to scalar encoding overhead
- Each SIMD comparison requires: memcpy, bswap, XOR, vector construction
- Encoding cost dominates any parallelism benefits

**Recommendation**: Use `SearchMode::Binary` for byte arrays. SIMD only benefits native primitive types (int32_t, uint32_t, int64_t, uint64_t, float, double).

## btree<Key, Value, LeafNodeSize, InternalNodeSize, Compare, SearchMode, Allocator>

**Note**: MoveMode parameter was removed - compiler automatically uses AVX instructions.

### Key Implementation Patterns

#### Bulk Transfer Operations (O(n²) → O(n))
```cpp
// ❌ Element-by-element (shifts array N times)
for (auto& elem : source) dest.insert(elem.first, elem.second);

// ✅ Bulk transfer (shifts array once)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kressler/fast-containers](https://github.com/kressler/fast-containers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
