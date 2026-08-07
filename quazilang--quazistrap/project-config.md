---
trigger: always_on
description: The primary reference for the Quazilang compiler (`qz`) — covers architecture, coding rules, language syntax, standard library, roadmap, and build pipeline.
---

# AGENTS.md — Quazilang Project Reference

The primary reference for the Quazilang compiler (`qz`) — covers architecture, coding rules, language syntax, standard library, roadmap, and build pipeline.

---

## Quick Commands

```bash
cargo build              # debug build
cargo build --release    # release build
cargo test               # run all 152+ unit tests
cargo clippy             # lint
cargo fmt                # format
```

CLI (dep: `clap 4.6`):
```bash
qz build <file> [-i|-c] [-o out] [-r] [-s] [--linker path]
qz run / qz check / qz fmt / qz clean
qz new <name> [--lib] / qz init [--lib]
qz debug [-i]
qz lsp
```

Output: `<stem>.qzi` (bytecode), `<stem>.o` (object), `<stem>`/`<stem>.exe` (binary).  
`.qzi` as input: skips frontend, goes straight to backend.  
Linker: `QUAZI_LINKER` env → `ld.lld` → `mold` → `ld` (Linux/macOS); `lld-link` → `link` (Windows). Linux uses `-dynamic-linker` and links `libc.so.6` / `libm.so.6` by full path to avoid GNU linker scripts that `ld.lld` cannot parse.  
`qz build myprog.o` — planned built-in linker path (P1).  
Rust edition 2024.

---

## Coding Rules

1. Write clean, maintainable, performant code.
2. Do not hardcode behavior that can be implemented directly in Quazilang.
3. Do not create useless attributes that contribute nothing — code that works without them is better.
4. Do not create excess intrinsics or attributes. Intrinsics are permitted when they are the cleanest or only viable choice; prefer stdlib code, but do not force awkward workarounds just to avoid an intrinsic.
5. Do not hardcode behavior.
6. Write all architectural changes to this file (AGENTS.md).
7. Aim not for the program just working, but for the program to be maintainable and clean.
8. No band-aid fixes. If a fix feels hacky, step back and redesign.
9. Keep the AST immutable after parsing; semantic analysis resolves meaning via annotations, not source mutation.
10. When fixing warnings, fix the root cause, not the symptom.

---

## Architecture

```
source → Loader → Lexer → Parser → Analyzer → Codegen → Backend (iced-x86) → .o → Linker → binary
```

QZI (`-b`): Codegen → serialized chunks, no backend.  
Object (`-c`): backend only, no linker.

### Component Guide

| Component | Path | Docs |
|-----------|------|------|
| Lexer | `src/lexer/` | [src/lexer/AGENTS.md](src/lexer/AGENTS.md) |
| Parser | `src/parser/` | [src/parser/AGENTS.md](src/parser/AGENTS.md) |
| Semantic | `src/semantic/` | [src/semantic/AGENTS.md](src/semantic/AGENTS.md) |
| Bytecode / Codegen | `src/bytecode/` | [src/bytecode/AGENTS.md](src/bytecode/AGENTS.md) |
| Backend (overview) | `src/backend/` | [src/backend/AGENTS.md](src/backend/AGENTS.md) |
| x86_64 Backend | `src/backend/x86_64/` | [src/backend/x86_64/AGENTS.md](src/backend/x86_64/AGENTS.md) |
| LSP | `src/lsp/` | [src/lsp/AGENTS.md](src/lsp/AGENTS.md) |
| Loader | `src/loader.rs` | (inline docs) |
| Project / manifest | `src/project.rs` | (inline docs) |

- The project is a single binary crate (`bin "qz"`) with inline `#[cfg(test)]` modules.
- No `tests/` integration directory yet — all tests are inline.

### Loader (`src/loader.rs`)

- `load_programs` — resolves imports recursively, merges dependency-first, parses as one `Program`.
- Std resolution: compiler `CARGO_MANIFEST_DIR/std` → `~/.quazi/std` / `%USERPROFILE%/.quazi/std`.
- `foo/mod.qz` = opaque module directory; `pub import` controls what's exported.
- Deduplicates via canonical-path `HashSet`. Circular imports safe.
- **Namespacing**: every non-entry file gets module-qualified function names (`bar.foo`). Entry files keep bare names.

### Project (`src/project.rs`)

- `quazi.toml`: `[package]`, `[build]`, `[dependencies]` (path + optional version). `quazi.lock` validated on build.
- `type = "lib"` → lib project; default entry `src/lib.qz`; default output `.qzi`.

---

## Language Quick Reference

```quazi
import std.io.stdout;
import std.io as io;
pub fn name[T](param: Type, ...rest: str) ReturnType {
    const x: i32 = 1 + 2;
    var y: &str = "hello";
    var n: u64 = 42 as u64;
    x += 1; x -= 1; x++; x--;
    // Bitwise operators
    var b: u32 = x & 0xFF;              // & bitwise AND
    b = x | 0x01;                       // | bitwise OR
    b = x ^ 0x0F;                       // ^ bitwise XOR
    b = x << 2;                         // << left shift
    b = x >> 1;                         // >> right shift (sign-preserving)
    // Logical operators
    var ok: bool = true && false;       // && logical AND
    ok = true || false;                 // || logical OR
    ok = !ok;                           // !  logical NOT
    if (cond) { ... } else { ... }
    for (cond) { ... }                  // while-loop
    for i : 0..10 { ... }              // range loop
    for i : collection { ... }         // iterator loop
    for i, v : collection { ... }      // index+value
    // break; continue;
    var arr = [1, 2, 3]; arr[0];
    ret expr;
}
unsafe fn ptr_fn(p: *u8) *u8 { ret p; }
unsafe { var x = ptr_fn(p); *x = 1; }

// Entry point may take no args or a single Array[str].
fn main(args: Array[str]) i32 { ret args.len() as i32; }

struct Foo[T] { field: T, const flag: bool, }

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quazilang/quazistrap](https://github.com/quazilang/quazistrap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-03 -->
