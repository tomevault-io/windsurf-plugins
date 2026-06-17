---
trigger: always_on
description: This file provides guidance to coding agents collaborating on this repository.
---

# AGENTS.md

This file provides guidance to coding agents collaborating on this repository.

## Mission

RiceVM is a Dis virtual machine and Limbo compiler in Rust.
The Dis VM is a register machine that executes bytecode compiled from the Limbo programming language.
The project includes a built-in Limbo compiler that compiles `.b` source files to `.dis` bytecode
without depending on the reference compiler from Inferno OS.
Priorities, in order:

1. Correct implementation of the Dis VM specification and Limbo language.
2. Clean, idiomatic Rust with safe abstractions over VM internals.
3. Clear separation of concerns across workspace crates.
4. Maintainable and well-tested code.

## Core Rules

- Use English for code, comments, docs, and tests.
- Keep `unsafe` usage minimal and well-documented; prefer safe Rust wherever possible.
- Never use `.unwrap()` or `.expect()` in non-test code (enforced by `make lint`). Production code should never panic.
- Prefer small, focused changes over large refactoring.
- Add comments only when they clarify non-obvious behavior.
- Do not add features, error handling, or abstractions beyond what is needed for the current task.
- Add tests for every bug fix and new feature to prevent regression.

## Writing Style

- Use Oxford commas in inline lists: "a, b, and c" not "a, b, c".
- Do not use em dashes. Restructure the sentence, or use a colon or semicolon instead.
- Avoid colorful adjectives and adverbs. Write "TCP proxy" not "lightweight TCP proxy", "scoring components" not "transparent scoring components".
- Use noun phrases for checklist items, not imperative verbs. Write "redundant index detection" not "detect redundant indexes".
- Headings in Markdown files must be in the title case: "Build from Source" not "Build from source". Minor words (a, an, the, and, but, or, for, in,
  on, at, to, by, of) stay lowercase unless they are the first word.

## Repository Layout

- `crates/ricevm-core/`: Shared types (Module, Opcode, Instruction, TypeDescriptor, and errors). No runtime logic.
- `crates/ricevm-loader/`: Binary format parser for `.dis` module files. One public function: `load(&[u8]) -> Result<Module, LoadError>`.
- `crates/ricevm-execute/`: Execution engine with 176 opcode handlers, heap, GC, built-in modules ($Sys, $Math, $Draw, $Tk, $Keyring, $Crypt, and
  audio),
  virtual device files, and file-based module loading.
- `crates/ricevm-limbo/`: Built-in Limbo compiler. Lexer, parser, code generator, and .dis binary writer.
  Compiles Limbo source to Dis bytecode end-to-end without depending on the reference limbo.dis compiler in the Inferno OS.
  Codegen is kind-aware: it tracks `NumKind` (word, big, and real) per local; sidecar maps for array element types, channel element types, and ADT
  layouts; and a fixup pass for forward-referenced calls and spawns.
- `crates/ricevm-cli/`: CLI with `run`, `dis`, and `compile` subcommands.
- `external/inferno-os/`: Git submodule of the Inferno OS repository (866 pre-compiled `.dis` files, Limbo source, and reference VM source in
  `libinterp/xec.c` for correctness validation).
- `Makefile`: GNU Make wrapper around `cargo` commands (`make test`, `make build`, `make lint`, etc.).
- `rust-toolchain.toml`: Pinned Rust toolchain (1.92.0) with `rustfmt`, `clippy`, and `rust-analyzer`.

## Architecture

### Crate Dependency Graph

```
ricevm-cli
├── ricevm-core
├── ricevm-loader → ricevm-core
└── ricevm-execute → ricevm-core and ricevm-loader
```

`ricevm-execute` depends on `ricevm-loader` for runtime module loading (the `load` opcode reads `.dis` files from disk).

### Key Internal Modules in `ricevm-execute`

| Module         | Purpose                                                                                          |
|----------------|--------------------------------------------------------------------------------------------------|
| `vm.rs`        | `VmState` struct, execution loop with cooperative threading, and thread suspend/resume           |
| `frame.rs`     | `FrameStack` with two-phase push (`alloc_pending` and `activate_pending`)                        |
| `heap.rs`      | `Heap` with reference counting, copy-on-write strings, and `ArraySlice` shared views             |
| `gc.rs`        | Mark-and-sweep garbage collector (scans frames, MP, and loaded module MPs)                       |
| `address.rs`   | Operand resolution with `ModuleMp` virtual ranges and `decode_virtual_addr`                      |
| `memory.rs`    | Typed read/write on byte buffers with bounds checking                                            |
| `data.rs`      | Module data (MP) initialization with type-aware elem sizes and nested arrays                     |
| `filetab.rs`   | Portable file descriptor table with in-memory pipe, virtual device files, and non-blocking stdin |
| `ops/`         | 176 instruction handlers organized by category                                                   |
| `sys.rs`       | Built-in `$Sys` module (43 functions with tuple return support, `%b`, `%u`, and `%.*`)           |
| `math.rs`      | Built-in `$Math` module (66 functions including linear algebra)                                  |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [habedi/ricevm](https://github.com/habedi/ricevm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
