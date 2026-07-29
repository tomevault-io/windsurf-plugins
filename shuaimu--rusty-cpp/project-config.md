---
trigger: always_on
description: This file is the repository guidance for coding agents. It is synced from
---

# Rusty-CPP Agent Guide

This file is the repository guidance for coding agents. It is synced from
`CLAUDE.md` and should be updated whenever the project context there changes.

Last synced from `CLAUDE.md`: January 2026 context.

## Project Overview

Rusty-CPP is a Rust-based static analyzer that applies Rust-style ownership,
borrowing, lifetime, and safe/unsafe rules to C++ code. The goal is to catch
memory-safety issues at compile time without runtime overhead.

- Supported C++ standard: strict C++23.
- Parser configuration: `-std=c++23`.
- CMake configuration: `CMAKE_CXX_STANDARD 23` and `CMAKE_CXX_EXTENSIONS OFF`.
- Default toolchain: clang/LLVM. CMake auto-selects `clang++-{19,18,17,16}`
  when `CMAKE_CXX_COMPILER` and `$CXX` are unset.
- GCC can build the non-module subset, but transpiled C++20 module targets are
  clang-only.

## Current Implementation State

The analyzer has broad test coverage and many implemented safety checks. Recent
January 2026 context says 670+ tests cover templates, variadic templates, STL
annotations, C++ casts, pointer safety, move detection, reassignment after move,
borrow checking, unsafe propagation, unsafe blocks, cross-function lifetime
checks, lambda capture safety, RAII tracking, partial moves/borrows, function
pointer safety, string literal tracking, STL use-after-move, and integration
behavior.

Recently implemented features include:

- Chained method temporary detection for patterns like
  `Builder().method().get_ref()`, using `@lifetime: (&'self) -> &'self`.
- Loop dangling reference detection for references to loop-local variables that
  escape an iteration.
- `rusty::move` and `rusty::copy` semantics, including Rust-like invalidation of
  mutable references and forbidden `std::move` on references in `@safe` code.
- Function pointer safety wrappers:
  `rusty::SafeFn`, `rusty::UnsafeFn`, `rusty::SafeMemFn`,
  `rusty::UnsafeMemFn`.
- String literal safety in `@safe` code.
- Partial borrow tracking for individual and nested struct fields.
- RAII tracking for references/pointers stored in containers, user-defined RAII
  types, iterator/container lifetime relationships, lambda escape analysis,
  member lifetimes, and `new`/`delete`.
- Conflict detection, transitive borrow tracking, two-state safety checking,
  header-to-implementation annotation propagation, template analysis, STL
  lifetime annotations, unified external annotations, and raw pointer safety.

Partially implemented or known incomplete areas:

- Virtual function calls have only basic method-call support.
- Loop counter variables declared in `for (int i = ...)` are not fully tracked in
  the variables map.
- Constructor initialization order is not checked.
- Exception handling is not modeled.
- Diagnostics do not yet include code snippets, fix suggestions, or detailed
  borrowing-rule explanations.
- No IDE/LSP integration.

## Safety Model

The project uses a two-state safety system:

- `@safe`
- `@unsafe`

Unannotated code is `@unsafe` by default. `@safe` code can only call `@safe`
functions directly. Calling `@unsafe`, unannotated, STL, or external functions
from `@safe` code requires an `@unsafe` block.

Annotations only attach to the next code element and can have suffixes, such as:

```cpp
// @safe-reviewed
void audited_function();

// @unsafe: uses raw pointers for performance
void low_level_function();
```

Annotation precedence is:

1. Function-level annotations.
2. Class-level annotations.
3. Namespace-level annotations.

Namespace annotations are per-file, not global. The same namespace may be marked
safe in one file and unsafe in another, which supports gradual migration.

## Lifetime Annotation Syntax

Lifetime annotations live in headers and use Rust-like syntax:

```cpp
// @lifetime: &'a
const int& getRef();

// @lifetime: (&'a) -> &'a
const T& identity(const T& x);

// @lifetime: (&'a, &'b) -> &'a where 'a: 'b
const T& selectFirst(const T& a, const T& b);

// @lifetime: owned
std::unique_ptr<T> create();

// @lifetime: &'a mut
T& getMutable();
```

External annotations combine safety and lifetime information:

```cpp
// @external: {
//   third_party::process: [unsafe, (const Data& d) -> Result]
//   sqlite3_column_text: [unsafe, (stmt* s, int col) -> const char* where s: 'a, return: 'a]
// }
```

External functions should be marked `[unsafe]` unless they have been explicitly
audited and intentionally marked `[safe]`.

## Project Structure

```text
src/
|-- main.rs                    # CLI handling and include path resolution
|-- parser/
|   |-- mod.rs                 # Parse orchestration
|   |-- ast_visitor.rs         # AST traversal and function call extraction
|   |-- annotations.rs         # Lifetime annotation parsing
|   `-- header_cache.rs        # Header signature caching
|-- ir/
|   `-- mod.rs                 # IR with CallExpr, Return, CFG
|-- analysis/
|   |-- mod.rs                 # Main analysis coordinator
|   |-- ownership.rs           # Ownership state tracking
|   |-- borrows.rs             # Basic borrow checking
|   |-- lifetimes.rs           # Original lifetime framework
|   |-- lifetime_checker.rs    # Annotation-based checking
|   |-- scope_lifetime.rs      # Scope-based tracking
|   |-- lifetime_inference.rs  # Automatic inference

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shuaimu/rusty-cpp](https://github.com/shuaimu/rusty-cpp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
