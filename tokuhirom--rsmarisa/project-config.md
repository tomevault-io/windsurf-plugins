---
trigger: always_on
description: This project is a Rust port of [marisa-trie](https://github.com/s-yata/marisa-trie), a static and space-efficient trie data structure library originally written in C++.
---

# rust-marisa Project Guidelines

## Project Goal

This project is a Rust port of [marisa-trie](https://github.com/s-yata/marisa-trie), a static and space-efficient trie data structure library originally written in C++.

The primary goal is to create a faithful Rust implementation that maintains compatibility with the original library's design and behavior, while leveraging Rust's safety features and idioms.

## Core Principles

### 1. Respect the Original Structure

- **Mirror the directory structure**: The Rust codebase should reflect the original C++ structure as closely as possible
  - `lib/marisa/` → `src/marisa/`
  - `lib/marisa/grimoire/` → `src/grimoire/`
  - `include/marisa/*.h` → public API modules in `src/lib.rs` or dedicated modules

- **Maintain logical organization**: Keep related functionality together as in the original
  - `grimoire/io/` → `grimoire/io/` (reader, writer, mapper)
  - `grimoire/trie/` → `grimoire/trie/` (louds-trie, tail, cache, etc.)
  - `grimoire/vector/` → `grimoire/vector/` (bit-vector, flat-vector, etc.)
  - `grimoire/algorithm/` → `grimoire/algorithm/` (sorting, etc.)

### 2. Track Source File Mapping

Each Rust module should clearly indicate which C++ file(s) it was ported from:

```rust
//! Ported from: lib/marisa/grimoire/trie/louds-trie.h
//! Ported from: lib/marisa/grimoire/trie/louds-trie.cc
//!
//! LOUDS (Level-Order Unary Degree Sequence) trie implementation.
```

For files that combine multiple C++ sources:

```rust
//! Ported from:
//! - include/marisa/trie.h
//! - lib/marisa/trie.cc
//!
//! Main trie interface.
```

### 3. Preserve Data Structures and Algorithms

- **Keep the same data structures**: Use Rust equivalents that maintain the same memory layout and behavior where possible
  - C++ `std::vector<T>` → `Vec<T>` or custom wrappers if specific behavior is needed
  - C++ bitfields and packed structures → Rust structs with appropriate field ordering
  - C++ templates → Rust generics

- **Maintain algorithmic approaches**: The core algorithms (LOUDS construction, search, etc.) should follow the same logic as the original

- **Preserve constants and magic numbers**: Keep the same configuration constants, thresholds, and tuning parameters used in the original

- **Document deviations**: When the Rust implementation must differ from C++ (e.g., for safety or idiomatic reasons), document why:

```rust
// Note: Original C++ uses raw pointer arithmetic here.
// Rust implementation uses safe indexing with the same logic.
```

### 4. Binary File Format Compatibility

**Critical Requirement**: Generated trie files must be binary-compatible with C++ marisa-trie.

- **Same serialization format**: The Rust implementation must produce files that can be read by C++ marisa-trie, and vice versa

- **Byte-level compatibility**: Ensure:
  - Same byte ordering (endianness handling)
  - Same padding and alignment
  - Same bit-packing schemes
  - Same integer sizes (use explicit types like `u32`, `u64` instead of `usize`)

- **Verification approach**:
  ```bash
  # Build trie with Rust implementation
  echo -e "app\napple\napricot" | cargo run --example marisa-build > rust.dic

  # Verify with C++ implementation
  echo "app" | marisa-lookup cpp.dic  # Should work
  echo "app" | marisa-lookup rust.dic # Should also work
  ```

- **Test cross-compatibility**: Create tests that verify:
  - Files created by C++ can be loaded by Rust
  - Files created by Rust can be loaded by C++
  - Both produce identical search results

### 5. Enable Tracking of Upstream Changes

- **Reference original commit hashes**: In CLAUDE.md or a PORTING.md file, record the marisa-trie commit that was used as the porting baseline

- **Mark porting status**: Track which files have been ported, which are in progress, and which are pending

- **Keep parallel structure**: Avoid premature refactoring that would make it difficult to diff against the original

### 6. Port Test Cases and Distinguish Origins

- **Port all test files**: Tests from `tests/` directory should be ported to Rust tests
  - `tests/base-test.cc` → `tests/base_test.rs` or module tests
  - `tests/trie-test.cc` → `tests/trie_test.rs`
  - `tests/vector-test.cc` → `tests/vector_test.rs`
  - etc.

- **Maintain test coverage**: Ensure the Rust version has equivalent or better test coverage

- **Use Rust testing conventions**: Convert C++ test macros to Rust's `#[test]` and assertion macros

- **Clearly distinguish test origins**: Tests must be clearly marked to indicate whether they are:
  1. **Ported from C++ original**: Tests directly ported from the C++ test suite
  2. **Rust-specific additions**: New tests added in the Rust version

**Marking Convention**:

```rust
// For tests ported from C++ marisa-trie:
#[test]
fn test_bit_vector_basic() {
    // Ported from: tests/vector-test.cc::TestBitVector
    let mut bv = BitVector::new();
    // ...
}

// For Rust-specific tests:
#[test]
fn test_bit_vector_rust_specific() {
    // Rust-specific: Test trait implementations
    let bv = BitVector::default();
    assert!(bv.is_empty());
}
```

**Why this is important**:
- Maintains traceability to original test coverage

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tokuhirom/rsmarisa](https://github.com/tokuhirom/rsmarisa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
