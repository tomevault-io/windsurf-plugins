---
trigger: always_on
description: > Standard agent instructions for the `zapcode` project.
---

# AGENTS.md

> Standard agent instructions for the `zapcode` project.
> Read this before writing any code.

---

## What this project is

**Zapcode** is a minimal, secure TypeScript subset interpreter written in Rust,
designed specifically to execute code written by AI agents. It is the TypeScript equivalent of
[pydantic/monty](https://github.com/pydantic/monty).

The core thesis: LLMs produce faster, cheaper, more reliable results when they write code instead of
making sequential tool calls. Zapcode makes that possible for TypeScript/JavaScript stacks without
containers, sandbox services, or running untrusted code directly on the host.

**What Zapcode can do:**
- Execute a safe subset of TypeScript — enough for an agent to express what it wants to do
- Block all host access by default: filesystem, env vars, network, `require`, `import`
- Expose host functions to the sandbox — only functions you explicitly register
- Snapshot VM state to bytes at external function call boundaries — resume later in any process
- Start in microseconds (~2 µs for simple expressions)
- Be called from Rust, TypeScript/JavaScript (napi-rs), Python (PyO3), or WebAssembly
- Enforce resource limits: memory, execution time, stack depth, allocation count

**What Zapcode cannot do (by design):**
- Access the standard library beyond a safe subset (`console`, `JSON`, `Math`, `Array`, `Object`, `Promise`)
- Use `import` / `require` / dynamic imports
- Access `process`, `globalThis`, `eval`, `Function()`, `setTimeout`/`setInterval`
- Use proxies, WeakMap/WeakRef, `Symbol`, or `with` statements
- Execute regular expressions (parsed but execution is a no-op)
- Use `var` declarations (use `let`/`const`)

---

## Repository layout

```
zapcode/
├── crates/
│   ├── zapcode-core/       # Parser, IR, bytecode compiler, VM, snapshot
│   │   ├── src/
│   │   │   ├── parser/      # oxc_parser integration — AST → ZapcodeIR
│   │   │   │   ├── mod.rs   # AST walker (oxc → IR)
│   │   │   │   └── ir.rs    # IR type definitions
│   │   │   ├── compiler/    # ZapcodeIR → Bytecode
│   │   │   │   ├── mod.rs   # Compiler logic
│   │   │   │   └── instruction.rs  # Instruction enum (~55 opcodes)
│   │   │   ├── vm/          # Stack-based bytecode executor
│   │   │   │   ├── mod.rs   # VM main loop, dispatch, ZapcodeRun entry point
│   │   │   │   └── builtins.rs  # Built-in functions (console, Math, JSON, etc.)
│   │   │   ├── value.rs     # Value enum — runtime type system
│   │   │   ├── snapshot.rs  # Serialize/deserialize mid-execution VM state
│   │   │   ├── sandbox.rs   # Resource limits and tracking
│   │   │   ├── error.rs     # ZapcodeError — all error types
│   │   │   └── lib.rs       # Public API re-exports
│   │   ├── tests/           # 14 test files, 214+ tests
│   │   └── benches/         # divan benchmarks
│   ├── zapcode-js/         # napi-rs bindings → @unchartedfr/zapcode npm package
│   ├── zapcode-py/         # PyO3 bindings → zapcode pip package
│   └── zapcode-wasm/       # wasm-bindgen target for browser/edge use
├── AGENTS.md                # This file
├── CLAUDE.md                # Claude Code-specific guidance (references this file)
├── Cargo.toml               # Workspace root
└── README.md                # User-facing docs with benchmarks
```

---

## Architecture overview

### Pipeline

```
TypeScript source → parser (oxc) → ZapcodeIR → compiler → Bytecode → VM → Result/Snapshot
```

### Parser

Uses **[oxc_parser](https://github.com/oxc-project/oxc)** — the fastest TypeScript parser in Rust.
The parser walks the oxc AST and emits `ZapcodeIR`. Unsupported syntax produces
`ZapcodeError::UnsupportedSyntax` with span information.

### Supported syntax subset

| Feature | Status |
|---|---|
| `const`, `let` declarations | ✅ |
| Functions (declarations, arrows, expressions) | ✅ |
| `async function`, `await` | ✅ |
| Classes (`constructor`, methods, `extends`, `super`, `static`) | ✅ |
| Generators (`function*`, `yield`, `.next()`) | ✅ |
| Control flow (`if`, `for`, `while`, `do-while`, `switch`, `for-of`) | ✅ |
| `try` / `catch` / `finally`, `throw` | ✅ |
| Closures with mutable capture | ✅ |
| Destructuring (object and array) | ✅ |
| Spread/rest operators | ✅ |
| Optional chaining `?.` | ✅ |
| Nullish coalescing `??` | ✅ |
| Template literals | ✅ |
| Type annotations, interfaces, type aliases | Stripped at parse time |
| String methods (30+) | ✅ |
| Array methods (25+) | ✅ |
| Math, JSON, Object, Array, Promise | ✅ |
| `import` / `require` / `eval` | ❌ sandbox violation |
| Regular expressions | Parsed, not executed |
| `var` declarations | ❌ not supported |
| Decorators, Symbol, WeakMap/WeakSet | ❌ not supported |

### Value system

```rust
pub enum Value {
    Undefined,
    Null,
    Bool(bool),
    Int(i64),
    Float(f64),
    String(Arc<str>),
    Array(Vec<Value>),
    Object(IndexMap<Arc<str>, Value>),
    Function(Closure),
    BuiltinMethod { object_name: Arc<str>, method_name: Arc<str> },
    Generator(GeneratorObject),
}
```

Arrays and objects are **value types** (owned `Vec<Value>` / `IndexMap`), not reference types.
Mutation works via `SetIndex`/`SetProperty` instructions that push the modified value back
onto the stack for store-back to the variable.

### VM


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheUncharted/zapcode](https://github.com/TheUncharted/zapcode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
