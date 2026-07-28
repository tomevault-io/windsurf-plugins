---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Development Commands

### Building
- `cargo build --release` - Build release version
- `git submodule init && git submodule update` - Initialize HHVM submodule (required for first build)
- `cd third-party/hhvm && git apply ../../hhvm-patch.diff && cd ../../` - Apply HHVM patches for WASM compilation
- Binary is created at `./target/release/hakana-default`

### Formatting
- `cargo fmt` - Run before committing

### Testing
- `cargo test --workspace --release` - Run Rust unit tests
- `cargo run --release --bin=hakana test --reuse-codebase tests` - Run all integration tests (recommended)
- `cargo run --bin hakana test <path-to-test-dir>` - Run individual test directory
- Test directories are organized under `tests/` with subdirectories for different test types
- **IMPORTANT**: Unused variable analysis only runs for tests in `tests/unused/` directory
- **IMPORTANT**: When working on scanning code (in `src/code_info_builder/`), use the `--no-cache` flag to ensure functions are always re-scanned: `cargo run --release --bin=hakana test --no-cache <path-to-test-dir>`

### Security Analysis
- `cargo run --bin hakana security-check <path>` - Run security/taint analysis mode
- When security analysis is enabled, no other analysis is performed

### Binaries
- Main binary: `hakana` (src/main.rs)
- Language server: `hakana-language-server` (src/lsp.rs)

## Architecture Overview

Hakana is a typechecker for Hack built in Rust, designed to complement HHVM's built-in typechecker with enhanced type inference and security analysis.

### Hack File Conventions
- **File Extensions**: Use `.hack` for Hack source files
- **Opening Tags**: Hack files in this project should NOT start with `<?hh` - the opening tag is omitted
- **File Format**: Files should start directly with Hack code (classes, functions, namespaces, etc.)

### Core Components

**Workspace Structure**: Multi-crate workspace with specialized modules:
- `src/analyzer/` - Core type analysis engine with expression/statement analyzers
- `src/code_info/` - Type definitions, AST structures, and codebase metadata
- `src/orchestrator/` - High-level analysis coordination and caching
- `src/cli/` - Command-line interface and test runners
- `src/language_server/` - LSP implementation
- `src/code_info_builder/` - AST scanning and initial type inference

**Analysis Architecture**:
- Expression analysis in `src/analyzer/expr/` with specialized analyzers for calls, assignments, binary operations
- Statement analysis in `src/analyzer/stmt/` covering control flow, loops, conditionals
- Reconciler system in `src/analyzer/reconciler/` for type narrowing and assertion handling
- Scope management in `src/analyzer/scope/` for tracking variable types through control flow

**Type System**:
- Union types (`t_union.rs`) and atomic types (`t_atomic.rs`) as core type representations  
- Type combination and expansion in `src/code_info/ttype/`
- Template/generic support in `src/code_info/ttype/template/`
- Type comparison logic in `src/code_info/ttype/comparison/`

#### TAtomic Type Representation
- `TAtomic` enum in `src/code_info/ttype/t_atomic.rs` represents different atomic type variants
- Common variants include `TTypeAlias`, `TGenericParam`, `TNamedObject`, `TInt`, `TString`, etc.
- Type aliases (including newtypes) are represented as `TAtomic::TTypeAlias` with:
  - `name: StrId` - The type alias name (e.g., `StrId::MEMBER_OF` for `HH\MemberOf`)
  - `type_params: Option<Vec<TUnion>>` - Generic type parameters
  - `as_type: Option<Box<TUnion>>` - The underlying type for the alias
  - `newtype: bool` - Whether this is a newtype (distinct type) vs transparent alias

#### Type Intersection and Reconciliation
- Type narrowing (e.g., `is` checks) triggers intersection logic in `src/analyzer/reconciler/assertion_reconciler.rs`
- `intersect_atomic_with_atomic()` function handles intersection of two atomic types using pattern matching
- Each pattern match handles specific type combinations (e.g., `TInt` ∩ `TString` = empty)
- For complex types like `MemberOf`, intersection must:
  1. Extract the inner value type (second type parameter: `type_params[1]`)
  2. Recursively intersect the inner type with the other atomic type via `intersect_union_with_atomic()`
  3. Reconstruct the wrapper type preserving original structure
- **Important**: Bidirectional patterns needed - both `(MemberOf, Other)` and `(Other, MemberOf)` cases
- **Implementation Pattern**:
  ```rust
  (TAtomic::TTypeAlias { name: StrId::MEMBER_OF, type_params: Some(params), .. }, _) => {
      // Intersect inner type (params[1]) with the other type, preserve params[0]
      intersect_union_with_atomic(..., &params[1], other_type, ...)
          .map(|intersected| TAtomic::TTypeAlias {
              name: StrId::MEMBER_OF,
              type_params: Some(vec![params[0].clone(), intersected]),
              // ... preserve other fields
          })
  }
  ```

**Security Analysis**:
- Taint analysis system in `src/code_info/data_flow/` 
- Tracks data flow from sources (user input) to sinks (dangerous operations)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [slackhq/hakana](https://github.com/slackhq/hakana) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
