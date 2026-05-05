---
trigger: always_on
description: This is a high-performance, zero-allocation JSON parser written in C. Key design principles:
---

# C JSON Parser - AI Coding Guidelines

## Architecture Overview
This is a high-performance, zero-allocation JSON parser written in C. Key design principles:
- **Memory Pool Allocation**: Uses static pools (`JSON_VALUE_POOL_SIZE`, `JSON_STACK_SIZE`) for O(1) allocation instead of `malloc`
- **Zero-Copy Parsing**: Primitives use `reference` structs pointing to original input string
- **Linked Lists**: Arrays and objects stored as singly-linked lists with `last` pointers for O(1) append
- **Assembly Optimizations**: Lookup tables in assembly for fast character classification (`whitespace_lookup.asm`, `hex_lookup.asm`)

## Build System
- **Ninja-based**: Platform-specific build files (`build.linux.ninja`, `build.osx.ninja`, `build.windows.ninja`)
- **Clang + LLD**: Compiler and linker for performance
- **Dual Modes**: Debug (C89, `-g`) and performance (C17, `-O3 -march=native -flto`)
- **Key Scripts**:
  - `./build-c-json-parser.sh [target]` - Build specific target (default: `perf-c-json-parser`)
  - `./test.sh [target]` - Build and run tests (default: `main`)
  - `./perf.sh [variant]` - Performance testing variants

## Code Patterns
- **Tagged Union**: `json_value` uses `type` field to determine active union member
- **Reference Structs**: `{const char *ptr; size_t len;}` for zero-copy strings/numbers
- **Error Handling**: `json_error` enum with descriptive strings via `json_error_string()`
- **Memory Management**: `json_reset()` for pool reuse, `json_cleanup()` for zero-fill reset
- **Assembly Integration**: Include `.asm` files in build, call as C functions

## Testing Conventions
- **Unity Framework**: Test files in `test/` directory using `TEST()` macro
- **Coverage-Driven**: Extensive test cases for edge cases and error paths
- **Performance Variants**: Test both string-validation and no-validation builds
- **Fuzzing-Ready**: Random JSON generation for comprehensive testing

## SIMD Architecture (AVX2 & SSE2)

### AVX2 (Advanced Vector Extensions 2)
- **Register Width**: 256-bit registers (`__m256i`) — processes 32 bytes per load
- **Platform Requirements**: Haswell (2013) or newer, AMD Excavator (2015) or newer
- **String Parsing**: Quote/escape scanning with `_mm256_cmpeq_epi8` (2× unrolled 32-byte chunks = 64 bytes/iteration)
- **String Validation**: Control-character detection (bytes < 0x20) via:
  - `_mm256_set1_epi8(0x80)` to create XOR bit-flip mask
  - `_mm256_xor_si256()` to shift comparison domain (converts unsigned to signed comparison)
  - `_mm256_cmpgt_epi8()` to perform `0x20_shifted > byte_shifted` check
  - Early return if `_mm256_movemask_epi8()` result is non-zero
- **Compilation**: Enabled automatically by `-march=native` on capable systems
- **Performance**: ~0.19 seconds average across 100 benchmark runs

### SSE2 (Streaming SIMD Extensions 2)
- **Register Width**: 128-bit registers (`__m128i`) — processes 16 bytes per load
- **Platform Requirements**: Pentium 4 (2000) or newer, all modern systems
- **String Parsing**: Quote/escape scanning with `_mm_cmpeq_epi8` (4× unrolled 16-byte chunks = 64 bytes/iteration)
- **String Validation**: Control-character detection (bytes < 0x20) via:
  - `_mm_set1_epi8(0x80)` to create XOR bit-flip mask
  - `_mm_xor_si128()` to shift comparison domain
  - `_mm_cmplt_epi8()` to perform `byte_shifted < 0x20_shifted` check (direct comparison direction)
  - Early return if `_mm_movemask_epi8()` result is non-zero
- **Semantics**: Equivalent to AVX2 (both reject bytes < 0x20), different comparison ops due to XOR inversion
- **Fallback When**: AVX2 unavailable; provides performance boost over scalar on all modern CPUs

### Scalar Fallback
- **When Used**: Platforms without SSE2 support (embedded ARM, MIPS, etc.) or if compiled without SIMD flags
- **Implementation**: Byte-by-byte C loop for quote/escape detection and validation
- **Performance**: ~10% slower than SIMD variants but still reasonable for non-time-critical applications

### String Validation Details
- **Compile-Time Flag**: `-DSTRING_VALIDATION` (default: enabled)
- **Purpose**: Reject JSON strings containing control characters (bytes 0x00–0x1F) per JSON specification
- **Test Coverage**: `test_validate_no_error` (test/test.c:6070–6088) validates all 32 control characters are rejected
- **Overhead**: ~8–10% performance cost when enabled vs disabled (benchmark-dependent)
- **Location in Code**: `parse_string()` function (src/json.c:220–530) performs validation at post-quote check and during quote-scan sequence

## Performance Considerations
- **Avoid Malloc**: Use memory pools (static arrays) for all JSON structures — achieves O(1) allocation with zero fragmentation
- **Lookup Tables**: Assembly-optimized character classification in `whitespace_lookup.asm` and `hex_lookup.asm`
- **SIMD Vectorization**: AVX2 (256-bit) for Haswell+, SSE2 (128-bit) fallback for Pentium 4+
- **Iterative Parsing**: `json_parse_iterative()` available for deep nesting to avoid stack overflow vs recursive `json_parse()`
- **String Validation**: Toggle via `-DSTRING_VALIDATION` compile flag (enabled by default, 8–10% overhead)
- **Profile-Guided Optimization (PGO)**: Build system supports PGO workflow for additional performance gains (~15–20% depending on workload)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [default-writer/c-json-parser](https://github.com/default-writer/c-json-parser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
