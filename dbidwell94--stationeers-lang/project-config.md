---
trigger: always_on
description: **Slang** is a high-level programming language that compiles to IC10 assembly for the game Stationeers. The compiler is a multi-stage Rust system with a C# BepInEx mod integration layer.
---

# Slang Language Compiler - AI Agent Instructions

## Project Overview

**Slang** is a high-level programming language that compiles to IC10 assembly for the game Stationeers. The compiler is a multi-stage Rust system with a C# BepInEx mod integration layer.

**Key Goal:** Reduce manual IC10 assembly writing by providing C-like syntax with automatic register allocation and device abstraction.

## Architecture Overview

### Compilation Pipeline

The compiler follows a strict 4-stage pipeline (in [rust_compiler/libs/compiler/src/v1.rs](rust_compiler/libs/compiler/src/v1.rs)):

1. **Tokenizer** (libs/tokenizer/src/lib.rs) - Lexical analysis using `logos` crate

   - Converts source text into tokens
   - Tracks line/span information for error reporting
   - Supports temperature literals (c/f/k suffixes)

2. **Parser** (libs/parser/src/lib.rs) - AST construction

   - Recursive descent parser producing `Expression` tree
   - Validates syntax, handles device declarations, function definitions
   - Output: `Expression` enum containing tree nodes

3. **Compiler (v1)** (libs/compiler/src/v1.rs) - Semantic analysis & code generation

   - Variable scope management and register allocation via `VariableManager`
   - Emits IL instructions to `il::Instructions`
   - Error types use `lsp_types::Diagnostic` for editor integration

4. **Optimizer** (libs/optimizer/src/lib.rs) - Post-generation optimization
   - Currently optimizes leaf functions
   - Optional pass before final output

### Cross-Language Integration

- **Rust Library** (`slang.dll`/`.so`): Core compiler logic via `safer-ffi` C FFI bindings
- **C# Mod** (`StationeersSlang.dll`): BepInEx plugin integrating with game UI
- **Generated Headers** (via `generate-headers` binary): Auto-generated C# bindings from Rust

### Key Types & Data Flow

- `Expression` tree (parser) → `v1::Compiler` processes → `il::Instructions` output
- `InstructionNode` wraps IC10 assembly with optional source span for debugging
- `VariableManager` tracks scopes, tracks const/device/let distinctions
- `Operand` enum represents register/literal/device-property values

## Critical Workflows

### Building

```bash
cd rust_compiler
# Build for both Linux and Windows targets
cargo build --release --target=x86_64-unknown-linux-gnu
cargo build --release --target=x86_64-pc-windows-gnu

# Generate C# FFI headers (requires "headers" feature)
cargo run --features headers --bin generate-headers

# Full build (run from root)
./build.sh
```

### Testing

```bash
cd rust_compiler
# Run all tests
cargo test --package compiler --lib

# Run specific test file
cargo test --package compiler --lib tuple_literals

# Run single test
cargo test --package compiler --lib -- test::tuple_literals::test::test_tuple_literal_size_mismatch --exact --nocapture
```

### Quick Compilation

!IMPORTANT: make sure you use these commands instead of creating temporary files.

```bash
cd rust_compiler
# Compile Slang code to IC10 using current compiler changes
echo 'let x = 5;' | cargo run --bin slang
# Compile Slang code to IC10 with optimization
echo 'let x = 5;' | cargo run --bin slang -z
# Or from file
cargo run --bin slang -- input.slang -o output.ic10
# Optimize the output with -z flag
cargo run --bin slang -- input.slang -o output.ic10 -z
```

## Codebase Patterns

### Test Structure

Tests follow a macro pattern in [libs/compiler/src/test/mod.rs](rust_compiler/libs/compiler/src/test/mod.rs):

```rust
#[test]
fn test_name() -> Result<()> {
    let output = compile!(debug "slang code here");
    assert_eq!(
      output,
      indoc! {
         "Expected IC10 output here"
      }
      );
    Ok(())
}
```

- `compile!()` macro: full pipeline from source to IC10
- `compile!(result ...)` for error checking
- `compile!(debug ...)` for intermediate IR inspection
- Test files organize by feature: `binary_expression.rs`, `syscall.rs`, `tuple_literals.rs`, etc.

### Error Handling

All stages return custom Error types implementing `From<lsp_types::Diagnostic>`:

- `tokenizer::Error` - Lexical errors
- `parser::Error<'a>` - Syntax errors
- `compiler::Error<'a>` - Semantic errors (unknown identifier, type mismatch)
- Device assignment prevention: `DeviceAssignment` error if reassigning device const

### Variable Scope Management

[variable_manager.rs](rust_compiler/libs/compiler/src/variable_manager.rs) handles:

- Tracking const vs mutable (let) distinction
- Device declarations as special scope items
- Function-local scopes with parameter handling
- Register allocation via `VariableLocation`

### LSP Integration

Error types implement conversion to `lsp_types::Diagnostic` for IDE feedback:

```rust
impl<'a> From<Error<'a>> for lsp_types::Diagnostic { ... }
```

This enables real-time error reporting in the Stationeers IC10 Editor mod.

## Project-Specific Conventions

### Tuple Destructuring

The compiler supports tuple returns and multi-assignment:

```rust
let (x, y) = func();  // TupleDeclarationExpression
(x, y) = another_func();  // TupleAssignmentExpression
```

Compiler validates size matching with `TupleSizeMismatch` error.

### Device Property Access


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dbidwell94/stationeers_lang](https://github.com/dbidwell94/stationeers_lang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
