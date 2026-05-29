---
trigger: always_on
description: This file provides guidance to Claude Code when working with the StoffelVM repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with the StoffelVM repository.

## Repository Overview

`StoffelVM` is a register-based virtual machine optimized for Multi-Party Computation (MPC). It executes bytecode compiled from StoffelLang, supporting both basic types (integers, booleans, strings) and complex types (objects, arrays, closures, foreign objects).

**Workspace crates:**
- `stoffel-vm` - The VM runtime and CLI
- `stoffel-vm-types` - Shared types (instructions, values, binary format)

**Primary consumers:** Stoffel CLI, all SDKs
**Bytecode source:** Stoffel-Lang compiler

## Development Commands

```bash
# Build the VM
cargo build
cargo build --release

# Build specific crate
cargo build -p stoffel-vm
cargo build -p stoffel-vm-types

# Run tests
cargo test
cargo test -p stoffel-vm
cargo test -p stoffel-vm-types

# Build the CLI runner
cargo build --release -p stoffel-vm

# Run a compiled program
./target/release/stoffel-run path/to/program.stfbin [entry_function]

# Format and lint
cargo fmt
cargo clippy

# Generate documentation
cargo doc --open
```

## Repository Structure

```
StoffelVM/
├── Cargo.toml                    # Workspace definition
├── README.md
├── CLAUDE.md
├── crates/
│   ├── stoffel-vm/              # VM runtime crate
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── main.rs          # CLI entry point (stoffel-run)
│   │       ├── lib.rs           # Library exports
│   │       ├── core_vm.rs       # VirtualMachine implementation
│   │       ├── functions.rs     # VMFunction definition
│   │       ├── activation.rs    # Call stack / activation records
│   │       ├── stdlib.rs        # Built-in functions
│   │       ├── hooks.rs         # Debug/instrumentation hooks
│   │       ├── ffi.rs           # Rust FFI bridge
│   │       └── net/             # MPC network integration
│   │           └── hb_engine.rs # HoneyBadger MPC engine
│   └── stoffel-vm-types/        # Shared types crate
│       ├── Cargo.toml
│       └── src/
│           ├── lib.rs           # Type exports
│           ├── instructions.rs  # Instruction enum
│           ├── core_types.rs    # Value enum
│           ├── compiled_binary/ # Binary format handling
│           └── utils/           # Serialization utilities
├── examples/                     # Example programs
└── benches/                      # Benchmark crates
```

## Architecture

### VM Execution Model

```
Bytecode Binary (.stfbin)
    ↓
Binary Loader (stoffel-vm-types)
    ↓ VMFunction[]
VirtualMachine (stoffel-vm)
    ↓
Instruction Execution Loop
    ↓
Result (Value)
```

### Core Components

| Component | File | Purpose |
|-----------|------|---------|
| VirtualMachine | `core_vm.rs` | Main VM state and execution loop |
| VMFunction | `functions.rs` | Function representation with instructions |
| Instruction | `instructions.rs` | VM instruction enum |
| Value | `core_types.rs` | Runtime value types |
| Activation | `activation.rs` | Call stack management |
| StdLib | `stdlib.rs` | Built-in functions |
| Hooks | `hooks.rs` | Debugging/instrumentation |
| HoneyBadgerMpcEngine | `net/hb_engine.rs` | MPC protocol integration |

### Instruction Set

**Memory Operations:**
- `LD(dest, offset)` - Load from stack
- `LDI(dest, value)` - Load immediate
- `MOV(dest, src)` - Move between registers
- `PUSHARG(reg)` - Push function argument

**Arithmetic:**
- `ADD`, `SUB`, `MUL`, `DIV`, `MOD`

**Bitwise:**
- `AND`, `OR`, `XOR`, `NOT`, `SHL`, `SHR`

**Control Flow:**
- `JMP(label)` - Unconditional jump
- `JMPEQ`, `JMPNEQ`, `JMPLT`, `JMPGT` - Conditional jumps
- `CMP(reg1, reg2)` - Compare registers
- `CALL(function)` - Function call
- `RET(reg)` - Return from function

### Value Types

```rust
pub enum Value {
    I64(i64), I32(i32), I16(i16), I8(i8),
    U64(u64), U32(u32), U16(u16), U8(u8),
    Float(F64),
    Bool(bool),
    String(String),
    Object(usize),
    Array(usize),
    Foreign(usize),
    Closure(Arc<Closure>),
    Unit,
    Share(ShareType, Vec<u8>),  // MPC secret shares
}
```

### Built-in Functions

| Function | Purpose |
|----------|---------|
| `print` | Console output |
| `create_object` | Create key-value object |
| `create_array` | Create array |
| `get_field` | Access object/array field |
| `set_field` | Set object/array field |
| `array_length` | Get array length |
| `array_push` | Append to array |
| `create_closure` | Create closure |
| `call_closure` | Invoke closure |
| `get_upvalue` | Get captured variable |
| `set_upvalue` | Set captured variable |
| `type` | Get value type as string |

## Key Files

### `crates/stoffel-vm/src/core_vm.rs`
Main VirtualMachine implementation:
- Register management
- Instruction execution loop
- Function dispatch
- Value operations

### `crates/stoffel-vm/src/net/hb_engine.rs`
HoneyBadger MPC engine integration:
- C FFI exports for SDK bindings
- MPC operation dispatch
- Network message handling

### `crates/stoffel-vm-types/src/instructions.rs`
Instruction enum defining all VM operations:
- Must stay in sync with Stoffel-Lang codegen
- Serialization for binary format

### `crates/stoffel-vm-types/src/core_types.rs`
Value enum and type definitions:
- Runtime value representation
- Type conversion utilities


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Stoffel-Labs/StoffelVM](https://github.com/Stoffel-Labs/StoffelVM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
