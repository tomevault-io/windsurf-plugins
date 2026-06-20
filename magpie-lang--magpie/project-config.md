---
trigger: always_on
description: Comprehensive guide for writing Magpie (.mp) programs, debugging compiler errors, and using the Magpie CLI. Designed for agents who have never seen Magpie before.
---


# Magpie Language Programming Guide

Use this skill whenever you write, debug, or review Magpie `.mp` programs.

---

## 0) Non-negotiable Rules

1. **Binary-first:** treat compiler diagnostics as ground truth.
   - Run `magpie explain <CODE>` for any error code you don't recognize.
   - Use `--output json` for machine-readable diagnostics.
2. **Diagnose by error code first**, then apply the smallest fix.
3. **Smallest reproducer first** -- one tiny `.mp` file, then grow.
4. **Change one dimension at a time** (syntax, then types, then ownership, then backend).
5. **Prove fixes with commands and exit codes.**

---

## 1) What is Magpie

Magpie is a compiled, SSA-based language with explicit ownership, ARC memory management, and first-class GPU support. Programs are written in a low-level IR-like textual format (CSNF).

Key characteristics:
- **SSA form** -- every value is defined once, used by name (`%x`, `%y`)
- **Explicit ownership** -- you manually `borrow.shared`, `borrow.mut`, `share`, `clone.shared`
- **ARC-managed heap** -- the compiler inserts reference counting; you manage ownership transitions
- **Block-structured** -- functions contain labeled basic blocks (`bb0:`, `bb1:`) with explicit terminators
- **No implicit conversions** -- types must match exactly; `const` suffixes must match declared types
- **5 GPU backends** -- SPIR-V, Metal (MSL), PTX (CUDA), HIP (AMD), WGSL (WebGPU)

How Magpie differs from familiar languages:
- There are no expressions or statements -- only SSA instructions and terminators
- Control flow uses `br`, `cbr`, `switch` -- not `if/else/for/while`
- Variables are SSA locals (`%name`) -- assigned exactly once
- Functions are `@name` -- always prefixed with `@`
- Types are `TName` -- user-defined types start with `T`
- Comments use `;` (not `//` or `#`); doc comments use `;;`

---

## 2) Module Structure

Every `.mp` file must have a strict header in exactly this order:

```mp
module <module.path>
exports { <exported symbols> }
imports { <import groups> }
digest "<hex string>"
```

**The order is mandatory.** Swapping any header line causes a parse error (`MPP0002`).

### Minimal template (copy-paste this to start any program)

```mp
module demo.main
exports { @main }
imports { }
digest "0000000000000000"

fn @main() -> i32 {
bb0:
  ret const.i32 0
}
```

### Exports

List functions (`@name`) and types (`TName`) the module makes visible:

```mp
exports { @main, @helper, TPoint }
```

### Imports

Group imports by source module:

```mp
imports { std.io::{@println}, util.math::{@sum, TVector} }
```

Use imported functions with their full qualified path in calls:

```mp
call_void std.io.@println { args=[%msg] }
```

### Digest

A hex string for content hashing. Use `"0000000000000000"` as a placeholder during development.

---

## 3) Functions

### Function kinds

```mp
fn @name(%param: Type) -> ReturnType { ... }
async fn @name(%param: Type) -> ReturnType { ... }
unsafe fn @name(%param: Type) -> ReturnType { ... }
gpu fn @name(%param: Type) -> unit target(msl) { ... }
```

### Parameters and return types

```mp
fn @add(%a: i64, %b: i64) -> i64 {
bb0:
  %r: i64 = i.add { lhs=%a, rhs=%b }
  ret %r
}
```

Parameters are SSA names (`%name: Type`). Return type follows `->`.

### Meta blocks (optional)

```mp
fn @compute(%x: i64) -> i64 meta {
  uses { @helper, @util }
  effects { io, alloc }
  cost { time=5, space=2 }
} {
bb0:
  %r: i64 = call @helper { x=%x }
  ret %r
}
```

### Basic blocks and terminators

Every function body contains one or more blocks. Each block has a label (`bbN:`) and ends with exactly one terminator.

```
bb0:              ; block label (bb followed by a number)
  <instructions>  ; zero or more SSA assignments or void ops
  <terminator>    ; exactly one: ret, br, cbr, switch, or unreachable
```

#### Terminators

| Terminator | Syntax | Description |
|---|---|---|
| `ret` | `ret %value` or `ret const.i32 0` | Return a value |
| `ret` (void) | `ret` | Return unit (for `-> unit` functions) |
| `br` | `br bb1` | Unconditional branch |
| `cbr` | `cbr %cond bb_true bb_false` | Conditional branch (`%cond` must be `bool`) |
| `switch` | `switch %val { case 0 -> bb1 case 1 -> bb2 } else bb3` | Multi-way branch |
| `unreachable` | `unreachable` | Mark unreachable code path |

#### Branching example

```mp
fn @abs(%x: i64) -> i64 {
bb0:
  %neg: bool = icmp.slt { lhs=%x, rhs=const.i64 0 }
  cbr %neg bb1 bb2

bb1:
  %zero: i64 = const.i64 0
  %r: i64 = i.sub { lhs=%zero, rhs=%x }
  ret %r

bb2:
  ret %x
}
```

#### Phi nodes (merging values from multiple predecessors)

```mp
fn @max(%a: i64, %b: i64) -> i64 {
bb0:
  %cond: bool = icmp.sgt { lhs=%a, rhs=%b }
  cbr %cond bb1 bb2

bb1:
  br bb3

bb2:
  br bb3

bb3:
  %result: i64 = phi i64 { [bb1:%a], [bb2:%b] }
  ret %result
}
```

`phi` selects a value based on which predecessor block was taken. **Borrow values cannot appear in phi nodes** (`MPO0102`).

---

## 4) Type System

### 4.1 Primitive types

| Category | Types |
|---|---|
| Signed integers | `i1 i8 i16 i32 i64 i128` |
| Unsigned integers | `u1 u8 u16 u32 u64 u128` |
| Floats | `f16 f32 f64 bf16` |
| Boolean | `bool` |
| Unit | `unit` |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [magpie-lang/magpie](https://github.com/magpie-lang/magpie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
