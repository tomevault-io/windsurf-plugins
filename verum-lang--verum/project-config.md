---
trigger: always_on
description: **Architectural invariant**: Verum's VBC interpreter (Tier 0) and
---

# Verum Language Platform

## CRITICAL: No libc in interpreter or AOT

**Architectural invariant**: Verum's VBC interpreter (Tier 0) and
AOT-compiled binaries (Tier 1) MUST NOT call into libc.  All
runtime functionality goes through:

* **Linux**: direct syscalls via `syscall` / `svc #0` instructions.
* **macOS**: libSystem.B.dylib only (Apple-required boundary, NOT
  libc in the glibc/musl sense).
* **Windows**: kernel32.dll + ntdll.dll only (no MSVC CRT, no UCRT).
* **FreeBSD**: direct syscalls.
* **Embedded**: bare-metal, no OS dependencies.

See `docs/architecture/no-libc-architecture.md` for the full
ruleset, verification procedure (`ldd` / `otool` / `dumpbin`), and
the remaining migration punch-list.

When emitting LLVM IR, every per-platform decision (syscall
numbers, sockaddr layout, errno-fn name, socket-option constants,
…) reads `module.get_triple()` — the **target** triple — never
host `#[cfg(target_os = "...")]` directives.  HOST gates miscompile
cross builds.  Helpers in
`crates/verum_codegen/src/llvm/target_triple.rs`
(`target_is_linux` / `target_is_darwin` / `target_is_windows` /
`target_is_aarch64` / `target_is_x86_64`) are the canonical
inspection API.

## CRITICAL: Verum Grammar Specification

**AUTHORITATIVE SOURCE**: `grammar/verum.ebnf` - The ONLY source of truth for Verum syntax.

Before writing or modifying ANY `.vr` file, you MUST verify syntax against `grammar/verum.ebnf`.

### Verum is NOT Rust! Key Differences:

| Rust Syntax (WRONG) | Verum Syntax (CORRECT) | EBNF Reference |
|---------------------|------------------------|----------------|
| `struct Name { ... }` | `type Name is { ... };` | `type_def` |
| `enum Name { A, B }` | `type Name is A \| B;` | `variant_list` |
| `trait Name { ... }` | `type Name is protocol { ... };` | `protocol_def` |
| `impl Name { ... }` | `implement Name { ... }` | `impl_block` |
| `impl Trait for T` | `implement Trait for T` | `impl_type` |
| `Box::new(x)` | `Heap(x)` | semantic types |
| `Vec<T>` | `List<T>` | semantic types |
| `String` | `Text` | semantic types |
| `#[derive(...)]` | `@derive(...)` | `attribute` |
| `#[repr(C)]` | `@repr(C)` | `attribute` |
| `use foo::bar` | `mount foo.bar` | `mount_stmt` |
| `crate` | `cog` | (module system) |

### Built-in Functions and Macros (NO `!` Syntax Anywhere)

Verum does NOT use Rust-style `!` suffix anywhere:

| Rust Syntax (WRONG) | Verum Syntax (CORRECT) | Category |
|---------------------|------------------------|----------|
| `println!("...")` | `print("...")` | I/O (built-in) |
| `format!("x={}", x)` | `f"x={x}"` | Format literal |
| `panic!("error")` | `panic("error")` | Control flow (built-in) |
| `assert!(cond)` | `assert(cond)` | Testing (built-in) |
| `assert_eq!(a, b)` | `assert_eq(a, b)` | Testing (built-in) |
| `unreachable!()` | `unreachable()` | Control flow (built-in) |
| `select!{...}` | `select { ... }` | Async expression |
| `join!(a, b)` | `join(a, b)` | Async function (built-in) |
| `matches!(x, P)` | `x is P` | Pattern test (is operator) |
| `my_macro!(...)` | `@my_macro(...)` | User-defined macro |

**Rule**: All compile-time constructs use `@` prefix: `@derive(...)`, `@const`, `@cfg`, `@sql_query(...)`.

### Reserved Keywords (v5.1)
Only 3 reserved: `let`, `fn`, `is`

### Type Definition Syntax
```verum
// Record type (like struct)
type Point is { x: Float, y: Float };

// Sum type (like enum)
type Option<T> is None | Some(T);
type Tree<T> is Leaf(T) | Node { left: Heap<Tree<T>>, right: Heap<Tree<T>> };

// Protocol (like trait)
type Iterator is protocol {
    type Item;
    fn next(&mut self) -> Maybe<Self.Item>;
};

// Newtype
type UserId is (Int);

// Unit type
type Marker is ();
```

### Rank-2 Polymorphic Function Types
```verum
// Regular function type (rank-1): caller chooses T
type Processor<T> is fn(T) -> T;

// Rank-2 function type: fn<R>(...) - function works for ALL R
// The quantified type parameters scope only within the function type
type Transducer<A, B> is {
    transform: fn<R>(Reducer<B, R>) -> Reducer<A, R>,
};

// Reducer used by transducers
type Reducer<A, R> is fn(R, A) -> R;

// Example: Stateful rank-2 transducer
type StatefulTransducer<A, B, S> is {
    initial_state: S,
    transform: fn<R>(Reducer<B, R>, &mut S) -> Reducer<A, R>,
};
```
Key difference: In `fn<R>(...)`, `R` is universally quantified inside the function type - the caller cannot choose `R`, the function must work for any `R`.

## Philosophy

**Core Principles:**
- **Semantic Honesty**: Types describe meaning (`List`, `Text`, `Map`), not implementation (`Vec`, `String`, `HashMap`)
- **No Magic**: All dependencies explicit via `using [...]`, no hidden state
- **Gradual Safety**: Three-tier references allow performance/safety tradeoff
- **Zero-Cost Abstractions**: CBGR enables memory safety at ~15ns overhead

## Critical Distinctions

### Context System vs Computational Properties

| Aspect | Context System (DI) | Computational Properties |
|--------|---------------------|-------------------------|
| **Purpose** | Runtime dependency injection | Compile-time side effect tracking |
| **Keywords** | `context`, `provide`, `using` | (inferred from code) |
| **Values** | Database, Logger, FS, etc. | Pure, IO, Async, Fallible, Mutates |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [verum-lang/verum](https://github.com/verum-lang/verum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
