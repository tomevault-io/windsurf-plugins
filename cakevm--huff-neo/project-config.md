---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Huff Neo is a production-ready compiler for the Huff language - a low-level programming language for developing highly optimized smart contracts that compile to EVM bytecode. This is a maintained fork of the archived huff-rs repository.

## Code Organization Requirements

### Import Statements
- **MUST**: All `use` statements must be placed at the top of the file with the other imports
- **NEVER**: Place `use` statements inside functions or methods
- This ensures consistency, makes dependencies clear at a glance, and improves compilation performance

### Documentation Strings
- **Behavior-focused**: Describe what the code accomplishes, not how it's implemented
- Focus on the "what" and "why" rather than the "how"
- Explain the purpose, inputs, outputs, and any important side effects
- Avoid implementation details that may change over time
- Example: "Resolves a label to its bytecode offset" instead of "Searches the label map and returns the index"

## Build and Development Commands

### Essential Commands
```bash
# Build the compiler
make build              # Debug build
make release           # Release build with optimizations
make maxperf           # Maximum performance build

# Testing
make test              # Run all tests
make test-doc          # Run documentation tests
cargo test --package <crate-name> --test <test-name>  # Run specific test
cargo test <test_function_name>  # Run specific test function

# Code Quality
make fmt               # Format code
make clippy            # Run clippy linter
make taplo             # Format TOML files
make deny-check        # Check dependencies for security/licensing issues

# Pre-release (run before PRs)
make pre-release       # Runs all checks: mdbook test, fmt, taplo, clippy, tests, deny-check

# Documentation
make doc               # Generate Rust documentation
mdbook serve           # Serve the user book locally
```

### Compiler Usage
```bash
# Compile a Huff file
cargo run -- <file.huff> -b              # Show bytecode
cargo run -- <file.huff> -r              # Show runtime bytecode
cargo run -- <file.huff> -a              # Generate artifacts
cargo run -- <file.huff> -o output.json  # Specify output file

# Run tests in Huff files
cargo run -- test <file.huff>            # Run all tests in file
cargo run -- test <file.huff> -m <name>  # Run specific test macro
```

## Architecture Overview

### Crate Structure

The compiler is organized as a Rust workspace with the following key crates:

1. **`bin/hnc`** - CLI interface, handles command-line arguments and orchestrates compilation
2. **`crates/lexer`** - Tokenizes Huff source code into tokens
3. **`crates/parser`** - Parses tokens into an Abstract Syntax Tree (AST)
4. **`crates/codegen`** - Generates EVM bytecode from the AST
5. **`crates/core`** - Core compiler logic, coordinates lexer/parser/codegen
6. **`crates/utils`** - Shared types, error handling, AST definitions
7. **`crates/test-runner`** - Executes Huff test macros using Anvil/REVM
8. **`crates/js`** - WASM bindings for JavaScript/browser usage

### Compilation Pipeline

1. **File Resolution** (`core/src/lib.rs`)
   - Resolves imports and includes
   - Flattens multiple files into a single source

2. **Lexing** (`lexer/src/lib.rs`)
   - Converts source text to tokens
   - Handles context-aware lexing (macro body vs. top-level)
   - Tracks source spans for error reporting

3. **Parsing** (`parser/src/lib.rs`)
   - Builds AST from tokens
   - Main types: `Contract`, `MacroDefinition`, `Statement`
   - Validates syntax and macro signatures

4. **Codegen** (`codegen/src/lib.rs`)
   - Walks the AST to generate bytecode
   - Key methods: `generate_main_bytecode()`, `generate_constructor_bytecode()`
   - Handles label resolution, jump tables, macro expansion

### Key Concepts

#### Label Scoping (Recently Updated)
- Each macro invocation has its own label scope to prevent overwriting
- Labels can be shadowed across different scope depths
- Label resolution order:
  1. Current scope and parent scopes (up the tree)
  2. Child scopes (down the tree) - allows parent to reference nested labels
  3. Sibling scopes (same parent) - fallback for cross-references
- Duplicate labels in siblings only error when cross-referenced
- Scope tracking uses `scope_depth` and `scope_path` (with invocation offsets for uniqueness)

#### Macro System
- **Inline macros**: Code is inserted at call site
- **Function macros** (outlined): Jump to shared code, return via jumpdest
- **Test macros**: Special macros executed by test runner
- Macros support parameters that are substituted during expansion

#### Built-in Functions
- `__FUNC_SIG()`, `__EVENT_HASH()`, `__ERROR()` - ABI encoding helpers
- `__RIGHTPAD()`, `__LEFTPAD()` - Byte padding (code tables only)
- `__BYTES()` - Raw bytes insertion
- `__TABLESIZE()`, `__TABLESTART()` - Jump table utilities
- `__CODESIZE()` - Get size of compiled macro

#### Important Files for Understanding Flow
- `crates/codegen/src/lib.rs` - Main bytecode generation logic
- `crates/codegen/src/irgen/statements.rs` - Statement-level code generation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cakevm/huff-neo](https://github.com/cakevm/huff-neo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
