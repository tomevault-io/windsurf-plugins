---
trigger: always_on
description: The project you are currently working on is called goml.
---

# Repository Guidelines

The project you are currently working on is called goml.

goml is a statically typed language with syntax similar to Rust, but it includes garbage collection (GC) and compiles to Go, so it does not require lifetime annotations and has no ownership system. In essence, it is closer in nature to OCaml or ML (Meta Language).

In goml, top-level functions must have fully explicit type signatures, and only top-level functions support generic type parameters. Generics are denoted using square brackets. Closures can be defined within functions, but they must have a single concrete type — goml does not include let-polymorphism (a.k.a. let-generalization). goml also supports defining traits, as well as enums and structs similar to Rust, and allows user-defined traits.

The generated Go code does not include generics — goml performs monomorphization by instantiating its own generic function calls. The generated code also does not contain Go closures — goml applies lambda lifting by performing lambda lifting (via lambda lifting) on its local functions.

The file extension for goml source files is `.gom`.

## Project Snapshot
- Go/Rust-inspired language frontend lowers through `lexer → parser → CST → AST → HIR → typed AST → Core -> Mono -> Lift → ANF` before emitting Go (`crates/compiler/src/go`).
- The CLI driver in `crates/goml/src/main.rs` handles both project-level commands and compiler subcommands; regression tests in `crates/compiler/src/tests` compare every IR stage and execute the Go output.
- The `webapp` folder hosts a Vite/React playground using Wasm bindings from `crates/wasm-app`; it can display each IR stage while execution is stubbed out.
- `typer/name_resolution.rs` should only handle AST → HIR lowering plus pure name resolution and visibility metadata; avoid making decisions that depend on `GlobalTypeEnv` or type information. Cross-module export filtering happens at package/interface construction time.
- `typer/check.rs` should only handle HIR → TAST type inference, checking, and constraint generation; avoid name-resolution responsibilities such as "fallback resolution paths" or cross-package name lookup.
- Any "ambiguity" should produce recoverable diagnostics; never `panic!` inside env/lookup. When multiple candidates exist, return `None` and report the error at a higher level.


## ANF IR and Join Points

The ANF (A-Normal Form) IR is the last intermediate representation before Go code generation. It lives in `crates/compiler/src/anf.rs` and is produced by the `anf_file` pass from the Lift IR.

### Core Data Types

```
Block { binds: Vec<Bind>, term: Term }
```
A block is a sequence of bindings followed by a terminal. This is the fundamental unit of ANF.

- **`Bind::Let(LetBind)`** — `let x = expr in ...` — binds a value expression to a local variable.
- **`Bind::Join(JoinBind)`** — `join k(params) { body } in ...` — declares a non-recursive join point (a local continuation).
- **`Bind::JoinRec(Vec<JoinBind>)`** — `joinrec { join k(params) { body } } in ...` — declares mutually recursive join points (used for loops).

Terminal terms (`Term`):
- **`Return(imm)`** — return a value from the function.
- **`Jump { target, args }`** — jump to a join point, passing arguments.
- **`If { cond, then_, else_ }`** — conditional branch; each branch is a `Block`.
- **`Match { scrut, arms, default }`** — pattern match; each arm body is a `Block`.
- **`Unreachable`** — marks dead code.

### What Are Join Points?

Join points are local continuations that represent "where control flow merges." They are the ANF equivalent of phi nodes in SSA, but are more structured — they have parameters (like function arguments) and a body.

**Key insight**: join points are NOT functions. They cannot escape their scope, are always called in tail position via `Jump`, and are guaranteed to be called exactly in the lexical scope where they are defined.

### How GoML Source Maps to ANF Join Points

**if-else expressions** (when used as expressions or with code after them):
```
// GoML source:
let result = if cond { a } else { b };
use(result)

// ANF:
join k(result) { use(result) } in    // k is the continuation after the if
if cond {
    jump k(a)                         // both branches jump to k
} else {
    jump k(b)
}
```

**Nested if-else** chains produce nested joins:
```
// GoML source:
let r = if c1 { "a" } else if c2 { "b" } else { "c" };

// ANF:
join k_outer(r) { ... } in
if c1 {
    jump k_outer("a")
} else {
    join k_inner(tmp) { jump k_outer(tmp) } in
    if c2 { jump k_inner("b") } else { jump k_inner("c") }
}
```

**match expressions** produce similar patterns — all arms jump to the same continuation join.

**while loops** lower to `JoinRec`:
```
// GoML source:
while ref_get(i) < limit {
    body;
}

// ANF:
join exit() { ... } in
joinrec {
    join loop() {
        let cond = ref_get(i) < limit in
        if cond {
            <body binds>
            jump loop()           // continue → self-jump
        } else {
            jump exit()           // break → jump to exit
        }
    }
} in
jump loop()
```

### Go Code Generation from ANF (`crates/compiler/src/go/compile.rs`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lijunchen/goml](https://github.com/lijunchen/goml) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
