---
trigger: always_on
description: This file provides guidance to Claude Code when working with the Stoffel-Lang repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with the Stoffel-Lang repository.

## Repository Overview

`Stoffel-Lang` is the compiler for the Stoffel programming language. It compiles `.stfl` source files to bytecode compatible with StoffelVM. The language features modern syntax inspired by Rust, Python, and JavaScript, with strong static typing and type inference.

**Crate name:** `stoffellang`
**Output formats:** Text bytecode (`.bc`), binary bytecode (`.stflb`/`.stfbin`)
**Primary consumers:** Stoffel CLI, all SDKs

## Development Commands

```bash
# Build
cargo build
cargo build --release

# Run compiler directly
./target/release/stoffellang path/to/source.stfl

# Compile to binary bytecode
./target/release/stoffellang -b path/to/source.stfl

# With optimization level (0-3)
./target/release/stoffellang -O2 path/to/source.stfl

# Print intermediate representations
./target/release/stoffellang --print-ir path/to/source.stfl

# Run tests
cargo test

# Format and lint
cargo fmt
cargo clippy

# Generate documentation
cargo doc --open
```

## Repository Structure

```
Stoffel-Lang/
├── Cargo.toml
├── README.md
├── CLAUDE.md
├── src/
│   ├── main.rs              # CLI entry point
│   ├── lib.rs               # Library exports
│   ├── lexer.rs             # Tokenization
│   ├── parser.rs            # AST construction
│   ├── ast.rs               # AST node definitions
│   ├── semantic.rs          # Semantic analysis
│   ├── symbol_table.rs      # Symbol table management
│   ├── core_types.rs        # Type system definitions
│   ├── codegen.rs           # Bytecode generation
│   ├── bytecode.rs          # Bytecode representation
│   ├── binary_converter.rs  # Binary format serialization
│   ├── register_allocator.rs # Register allocation
│   ├── compiler.rs          # Compilation orchestration
│   ├── errors.rs            # Error types and reporting
│   ├── suggestions.rs       # Error suggestions/hints
│   ├── ufcs.rs              # Uniform Function Call Syntax
│   ├── ufcs_tests.rs        # UFCS tests
│   └── ffi.rs               # C FFI exports
└── tests/                   # Test source files (.stfl)
```

## Architecture

### Compilation Pipeline

```
Source (.stfl)
    ↓
Lexer (lexer.rs)
    ↓ Tokens
Parser (parser.rs)
    ↓ AST
Semantic Analysis (semantic.rs)
    ↓ Typed AST + Symbol Table
Code Generation (codegen.rs)
    ↓ Bytecode
Binary Converter (binary_converter.rs)
    ↓
Output (.stflb / .bc)
```

### Key Components

| Component | File | Purpose |
|-----------|------|---------|
| Lexer | `lexer.rs` | Tokenizes source code |
| Parser | `parser.rs` | Builds Abstract Syntax Tree |
| AST | `ast.rs` | AST node type definitions |
| Semantic | `semantic.rs` | Type checking, validation |
| Symbol Table | `symbol_table.rs` | Scope and symbol management |
| Codegen | `codegen.rs` | Generates VM instructions |
| Register Allocator | `register_allocator.rs` | Register assignment |
| Binary Converter | `binary_converter.rs` | Serializes to binary format |

### Type System

The compiler uses `stoffel-vm-types` for VM-compatible types:
- `i32`, `i64` - Integers
- `f32`, `f64` - Floating point
- `bool` - Boolean
- `string` - Strings
- Arrays and Objects
- Function types

## Key Files

### `src/main.rs`
CLI entry point with argument parsing:
- `-b` / `--binary` - Output binary format
- `-O<level>` - Optimization level (0-3)
- `--print-ir` - Print intermediate representations
- `-o` / `--output` - Output file path

### `src/codegen.rs`
Bytecode generation from typed AST:
- Translates expressions to VM instructions
- Manages constant pool
- Handles function definitions
- Generates entry points

### `src/binary_converter.rs`
Binary format serialization:
- Converts bytecode to `.stflb` format
- Compatible with StoffelVM binary loader
- Includes function metadata, constants

### `src/semantic.rs`
Semantic analysis:
- Type checking and inference
- Scope validation
- Function resolution
- Error collection

### `src/ffi.rs`
C FFI exports for SDK integration:
- Compile source from string/file
- Get bytecode output
- Error handling across FFI boundary

## API Contracts

### With StoffelVM

The compiler outputs instructions from `stoffel-vm-types`:
```rust
use stoffel_vm_types::Instruction;
use stoffel_vm_types::Value;
```

Instructions must match VM's instruction set:
- Memory: `LD`, `LDI`, `MOV`, `PUSHARG`
- Arithmetic: `ADD`, `SUB`, `MUL`, `DIV`, `MOD`
- Bitwise: `AND`, `OR`, `XOR`, `NOT`, `SHL`, `SHR`
- Control: `JMP`, `JMPEQ`, `JMPNEQ`, `JMPLT`, `JMPGT`, `CALL`, `RET`
- Compare: `CMP`

### With SDKs

SDKs use the compiler via:
1. **Library crate** - Direct Rust API
2. **FFI** - C bindings for Python, TypeScript

```rust
// Rust SDK usage
use stoffellang::Compiler;

let compiler = Compiler::new();
let bytecode = compiler.compile(source)?;
```

## Common Tasks

### Adding a New Language Feature

1. Update lexer in `lexer.rs` for new tokens
2. Add AST nodes in `ast.rs`
3. Update parser in `parser.rs`
4. Add type checking in `semantic.rs`
5. Generate bytecode in `codegen.rs`
6. Add tests in `tests/`

### Adding a New Instruction

1. Ensure instruction exists in `stoffel-vm-types`
2. Update `codegen.rs` to emit the instruction

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Stoffel-Labs/StoffelVM](https://github.com/Stoffel-Labs/StoffelVM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
