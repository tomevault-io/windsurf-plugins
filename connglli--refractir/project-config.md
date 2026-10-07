---
trigger: always_on
description: + Knowledge Background: Optimizing Compilers, Program Analysis, Symbolic Execution, SMT (Bit-Vector Theory)
---

# RefractIR: Agent Guideline

+ Knowledge Background: Optimizing Compilers, Program Analysis, Symbolic Execution, SMT (Bit-Vector Theory)
+ Implementation Language: C++ 20
+ Primary Source Directory: ./src


## Project Overview

RefractIR (internally called SymIR) is a **CFG-based symbolic intermediate representation** designed for:

- **Program synthesis**
- **Symbolic execution**
- **Constraint generation for SMT solvers (Bit-Vector logic)**

A RefractIR program is a **template**, not a fully concrete program. It may contain **symbols** (unknowns, marked with `?`) whose values are solved later by an SMT solver under constraints derived from:

1. A **specific execution path** through the CFG
2. Explicit **properties** (e.g., `require` and/or other user-provided properties) that must hold on that path

The key design goals are:

- SMT-friendliness (predictable BV constraints, minimal nonlinearity)
- Clear semantics with **strict undefined behavior (UB)**
- Explicit control-flow (CFG, no SSA, user-friendly)
- Simplicity and analyzability over expressiveness

RefractIR deliberately restricts expressions (flat, left-to-right, no parentheses) and uses **LLVM-style syntax** (`@`, `%`, `br`, basic blocks) while remaining language-agnostic.

The current formal specification is **[./docs/SPEC_v0.2.2.md](./docs/SPEC_v0.2.2.md)** (v0.2.2, complete and released — adds function calls (`call`) and interprocedural execution; external declarations (`decl`) with `-I` link resolution and behavioral contracts (`pre`/`post`/`ret`); and standard intrinsics, documented per-intrinsic in [./docs/intrinsics.md](./docs/intrinsics.md)). The v0.2.1 spec (aggregate pointers `ptr [N] T`, `ptr @S`, `ptrindex`, `ptrfield`; SIMD vector types `<N> T` with lane-wise arithmetic and `cmp`; floating-point value model; mask-based `select`) is preserved at [./docs/SPEC_v0.2.1.md](./docs/SPEC_v0.2.1.md), the v0.2.0 pointer baseline at [./docs/SPEC_v0.2.0.md](./docs/SPEC_v0.2.0.md), and the pre-pointer baseline at [./docs/SPEC_v0.1.0.md](./docs/SPEC_v0.1.0.md) for reference.

## Language at a Glance

Key characteristics:

- **Non-SSA**, mutable locals via `let mut`
- **Symbols**: solver-chosen unknowns (`@?x`, `%?y`)
- **CFG-based**: explicit basic blocks (`^label`)
- **Expressions**:
  - Flat, left-to-right
  - `+`, `-` at expression level
  - `*`, `/`, `%` at atom level
- **Division/modulo**: round toward 0 (C / SMT `bvsdiv`, `bvsrem`)
- **select** expression:
  - Lazy (only selected arm evaluated)
  - Expression-level conditional; mask-based `select <N> i1` for per-lane blends (v0.2.1)
- **Floating-point**:
  - `f32` (IEEE 754 single) and `f64` (IEEE 754 double)
  - Finite-only domain: ±∞ and NaN are UB
  - `%` is C's `fmod` (truncated-quotient remainder)
  - All operations use RNE rounding
- **Pointers (v0.2.0)**:
  - `ptr T` type, scalar/pointer pointees
  - `addr lv` produces `ptr T` (requires `let mut` root)
  - `load p`, `store p, v` for read/write through pointers
  - `null` literal (typed by context)
  - Pointer arithmetic: `ptr T ± iN → ptr T`, `ptr T - ptr T → i64` (element distance)
  - No `sym` of pointer type; no pointer/integer casts
- **Aggregate pointers (v0.2.1)**:
  - `ptr [N] T` / `ptr @S` — pointers to arrays and structs
  - `ptrindex <ptr [N] T>, <idx>` → `ptr T` (navigate array element)
  - `ptrfield <ptr @S>, <field>` → `ptr FieldType` (navigate struct field)
  - Packed `sizeof(@S) = Σ sizeof(field_i)`; no padding
  - No aggregate `load`/`store`; navigate to scalar/pointer leaves first
- **Vector types (v0.2.1)**:
  - `<N> T` — fixed-width SIMD vector of scalar elements
  - Lane-wise arithmetic: all scalar operators lift to vectors
  - `cmp <relop> lhs, rhs` — reified comparison → `i1` (scalar) or `<N> i1` (vector mask)
  - Lane access via subscript: `%v[%i]` reads/writes lane `i`
  - Whole-vector copy: `%v = %w;`
  - Not addressable: no `ptr <N> T`, no `addr` on vector locals
  - `sym` of vector type allowed (per-lane independent symbols)
- **Strict UB**:
  - Division/modulo by zero
  - Out-of-bounds array access
  - Reading `undef`
  - Null/uninitialised/out-of-bounds pointer dereference, cross-object pointer arithmetic or relational comparison
  - FP overflow/NaN, float-to-integer out-of-range
  - Vector lane-wise UB, out-of-bounds lane access (v0.2.1)

## Toolchain Overview

| Tool | Role |
|------|------|
| `symirc` | Translate `.sir` to C / WebAssembly |
| `symiri` | Interpret `.sir` programs |
| `symirsolve` | Concretize symbolic programs using SMT |

Documentation of each tool: [./docs/](./docs).

## Compilation / Analysis Pipeline

### Shared Frontend Pipeline

```
Source (.sir)
  ↓
Lexer
  ↓
Parser → AST
  ↓
CFG Builder
  ↓
TypeChecker (BV-aware)
  ↓
Semantic Checker
```

### Tool-Specific Pipelines

`symirc`:
```
Checked AST + CFG
  ↓
Lowering
  ↓
C / WASM Code Generation
```

`symiri`:
```
Checked AST + CFG
  ↓
Symbol Binding (--sym)
  ↓
Interpreter Execution
```

`symirsolve`:
```
Checked AST + CFG
  ↓
Path-based Symbolic Execution
  ↓
SMT Solving
  ↓
Concrete .sir
```

### 1. Lexer
- Converts source text into tokens
- Handles identifiers with sigils (`@`, `%`, `@?`, `%?`, `^`)
- Handles comments and string literals
- No semantic knowledge

### 2. Parser

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [connglli/RefractIR](https://github.com/connglli/RefractIR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
