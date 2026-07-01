---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A JVM implementation written in Rust — a learning project aiming for a complete JVM with LLVM JIT and GC. Actively being refactored, supports JDK 9+ (JImage class loading).

## Workspace Structure

```
jvm/                     # Binary crate — CLI entry point (main.rs + options.rs)
crates/
  classfile/             # JVM class file format type definitions (zero dependencies)
  class-parser/          # Parser: bytes -> ClassFile (Cursor + Read)
  vm/                    # Core VM: interpreter, Oop model, native methods, threading, JIT
  class-verification/    # Class verification (skeleton)
tools/
  javap/                 # Class file disassembler tool
tests/
  java/                  # Java test suite — 17 files covering arithmetic, OOP, arrays, exceptions, etc.
scripts/
  dev.sh                 # Convenience script for common tasks
documents/
  jvm-implementation-challenges.md  # Rust vs C++ JVM implementation difficulties
```

### Dependency graph

```
jvm (binary) → vm → class-parser → classfile
                          ↘ classfile
javap (tool) → class-parser → classfile
                  ↘ classfile
```

### Dependency management

All dependencies are centralized in the root `Cargo.toml` under `[workspace.dependencies]`.
Sub-crates reference them with `dep.workspace = true`. To add or upgrade a dependency,
edit only the root `Cargo.toml`.

## Key Commands

```bash
# Quick reference — use dev.sh for convenience
./scripts/dev.sh build              # Build workspace (debug)
./scripts/dev.sh build-release      # Build workspace (release)
./scripts/dev.sh run <Class> [args] # Run a Java class
./scripts/dev.sh test               # Run all tests
./scripts/dev.sh javap <classfile>  # Disassemble a class file
./scripts/dev.sh clean              # Clean build artifacts

# Or use cargo directly
cargo build                   # Builds all workspace members
cargo build --workspace       # Same as above (explicit)
cargo test --workspace        # Runs ALL tests: unit + class-parser + Java integration
cargo test -p class-parser hello_world
cargo run -p jvm -- <ClassName>
cargo run -p javap -- <classfile>
```

### Class parser test fixtures

Java source files live in `tests/java/src/` and are compiled to `.class` at build time via `build.rs` in both `class-parser` and `java-tests` packages. Do **not** commit `.class` files — only `.java` sources.

### Java integration tests

`tests/` is a workspace member (`java-tests` package) that runs 17 Java files against the JVM binary via `cargo test --workspace`. The `build.rs` compiles `.java` → `.class`, and `tests/tests/java_integration.rs` runs each class through the JVM.

## Architecture

### classfile (`crates/classfile/`)

Pure type definitions for the JVM class file format. No parsing logic. Core types:
- `ClassFile` — top-level struct
- `ConstantPool` / `ConstantPoolType` — constant pool entries
- `MethodInfo` / `FieldInfo` — method and field descriptors
- `Attribute` variants — Code, LineNumberTable, Exceptions, etc.
- `Opcode` — all 200+ JVM bytecode instructions

### class-parser (`crates/class-parser/`)

Parses raw bytes into `ClassFile`. Uses `std::io::Cursor` + `Read` (no nom). Files:
- `reader.rs` — `Reader` struct with `read_u8/u16/u32/bytes/utf8`
- `constant_pool.rs` — constant pool parsing
- `attributes.rs` — attribute parsing (Code, Signature, etc.)
- `fields.rs` / `methods.rs` — field/method parsing
- `class.rs` — top-level `parse_class_file` assembly
- `signature.rs` — method/field signature parsing (generics, arrays, etc.)

Entry point: `parse(&[u8]) -> Result<ClassFile>` or `parse_class` (alias).

### vm (`crates/vm/`)

The core VM. Major modules:

**oop/** — Object model
- `mod.rs` — `Oop` enum, slot-based heap access (`with_heap`/`with_heap_mut`)
- `heap.rs` — slot-based object allocation (`Oop::Ref(u32)`)
- `reference.rs` — `RefKind`, `RefKindDesc`, safe `Monitor` (std Mutex+Condvar)
- `class.rs` / `inst.rs` / `ary.rs` / `field.rs` — object kinds

**runtime/** — Execution engine
- `interp/` — interpreter, split into per-opcode files (no macros):
  - `const_ops.rs` / `load_store.rs` / `arith_ops.rs` / `stack_ops.rs` / `conversion.rs`
  - `control_flow.rs` / `compare.rs` / `object_ops.rs` / `field_ops.rs` / `array_ops.rs`
  - `monitor_ops.rs` / `exception.rs` / `read.rs`
- `jit/` — LLVM JIT compiler (inkwell):
  - `mod.rs` — JIT compiler lifecycle (thread-local), `Box::leak` boundedness test
  - `builder.rs` — bytecode → LLVM IR translation (~155/202 opcodes)
  - `ops.rs` — runtime callouts for complex operations (new, field access, arrays, ldc)
  - `runtime.rs` — JIT invoke context (TLS `JitInvokeCtx`), `invoke*` dispatch
- `frame.rs` / `stack.rs` / `slot.rs` / `local.rs` — call stack structures
- `invoke.rs` — method invocation logic (JIT first, interpreter fallback)
- `method.rs` — method representation
- `class_loader.rs` — class loading from classpath
- `class_path_manager.rs` — classpath management (DIR, JAR, JImage)
- `thread/` — Java threads, mutex, condvar, thread pool
- `vm.rs` — VM initialization and lifecycle

**native/** — JNI native method implementations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [douchuan/jvm](https://github.com/douchuan/jvm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
