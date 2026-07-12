---
trigger: always_on
description: A compiler for a functional programming language with pure/impure function tracking, algebraic data types, and pattern matching.
---

# blorp

A compiler for a functional programming language with pure/impure function tracking, algebraic data types, and pattern matching.

## Language Principles

The priorities of the language, in order: **safe, understandable, expressive, simple, fast, easy to learn,
predictable for tools to generate and humans to debug.**

All work on the compiler, standard library, documentation, and examples should uphold these principles.

### Safety

**1. No runtime panics — operations succeed by design.**
Division by zero returns 0, integer overflow wraps, there is no null. The runtime should never
surprise you with a crash. Reserve `Option`/`Result` for genuinely fallible operations. Infrastructure
exceptions (OOM, fiber stack overflow) are acceptable but language-level operations must be infallible.

**2. Value semantics — no shared mutable state, no cyclic data.**
Assignment copies. Closures capture by value. Record update creates new records. ARC works without
a cycle collector because cycles are structurally impossible. This is the architectural foundation
that makes everything else work — thread safety, deterministic memory, local reasoning.

**3. Thread-safe by default.**
Atomic reference counts. Value semantics. Channels for communication. The language does not give you
shared mutable references. You don't opt into thread safety — it's the only option.

### Understandability

**4. Purity tracking — the compiler tells you what has side effects.**
`pure func` is enforced by the compiler. Pure functions cannot call impure functions. Local mutation
is allowed in pure functions (pragmatic — it doesn't affect determinism). This enables safe
parallelization, caching, and equational reasoning.

**5. Immutability by default.**
`x = 5` is immutable. `var x = 5` is explicit opt-in to mutation. You see at a glance what can change.
Closures cannot capture mutable variables.

**6. Pattern matching with exhaustiveness checking.**
The compiler tells you when you've missed a case. Impossible states become compile errors.
Pattern matching is the primary control flow mechanism for conditional logic.

### Expressiveness

**7. Expressions over statements — everything returns a value.**
`if` and `match` are expressions. The last expression in a function body is the return
value. `?=` provides explicit Option/Result propagation without exceptions.

**8. UFCS for composition — any function is a method.**
`x.f(args)` desugars to `f(x, args)`. Enables left-to-right method chaining without OOP.
This is the primary composition mechanism — it replaces pipeline operators, method chains,
and nested function calls with a single, readable syntax.

**9. Traits for polymorphism — no class inheritance.**
Operator overloading via `Addable`/`Equatable`/etc. Generic bounds via `T: Orderable`.
The type system is flat and predictable. Trait method imports should work for any type
that implements the trait, not just the module they were imported from.

### Simplicity

**10. Minimize ceremony — don't force unwrapping things that can't fail.**
`length` returns `Int` not `Option[Int]`. If an operation can be made infallible by design,
do that instead of forcing error handling. The goal is less boilerplate, not less safety.

**11. Readable syntax — indentation-based, keyword operators, minimal noise.**
`and`/`or`/`not` instead of symbols. Colon + indent for blocks. Braces for records/dicts/vectors
(not for control flow). The code should read close to pseudocode.

### Speed

**12. Deterministic resource management — ARC, no GC.**
No garbage collector. No GC pauses. Predictable performance. Objects are freed when their reference
count drops to zero. COW makes "immutable" collections fast when uniquely owned.

**13. Compiles to C — native performance, any platform.**
The compilation target is the performance strategy. SIMD for vectors, direct C interop via
`foreign func`, and the entire C optimizer toolchain. The generated C should be clean enough
for the C compiler to optimize well.

**14. Compile-time safety with graduated escape hatches.**
Tensor dimensions verified statically. Range types (`..#N`) for proven-safe indexing with
zero runtime cost. `get()` for runtime-checked access when compile-time proof isn't possible.
Static where we can, dynamic where we must, infallible by default.

### Easy to Learn / Tool-Friendly

**15. Structured concurrency.**
`concurrent:` blocks auto-join all spawned work. No orphaned tasks. `detach` for explicit
fire-and-forget. Simple mental model that's hard to misuse.

**16. Clean C interop.**
`foreign func` is one declaration. Any C library is accessible. Low barrier to extending
the language with existing ecosystems.

### For Agents

When making decisions about the language, compiler, or standard library, use these principles
as a tiebreaker:

- If a change makes the language safer but more verbose, prefer safety (principles 1-3).
- If a change makes the language more expressive but harder to understand, prefer understandability (principles 4-6).
- If a change improves performance but adds complexity, prefer simplicity (principles 10-11) unless the performance gain is substantial.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kablorp/blorp](https://github.com/kablorp/blorp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
