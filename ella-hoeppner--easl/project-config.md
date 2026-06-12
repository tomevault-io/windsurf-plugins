---
trigger: always_on
description: Easl (Enhanced Abstraction Shader Language) is a Lisp-like shader language that compiles to WGSL. It uses S-expression syntax (parenthesized prefix notation) and provides generics, sum types (enums), higher-order functions, and expression-based control flow on top of what WGSL offers.
---

# Easl Compiler

Easl (Enhanced Abstraction Shader Language) is a Lisp-like shader language that compiles to WGSL. It uses S-expression syntax (parenthesized prefix notation) and provides generics, sum types (enums), higher-order functions, and expression-based control flow on top of what WGSL offers.

## Build & Test

```bash
cargo test --features window                        # run all tests (ALWAYS use this flag)
cargo test --features window <test_name>            # run a specific test
cargo run                                           # runs benchmark (compiles all .easl files in data/gpu/)
```

> **IMPORTANT**: Always pass `--features window` when running tests. Without it, the interpreter, wgpu GPU execution, and windowing code are all compiled out, causing most buffer/cpu/window tests to silently produce wrong results or be skipped. The `window` feature is required for any test that involves the interpreter, GPU compute/render, or the `IOManager` trait.

## Project Structure

- `src/lib.rs` — public API: `compile_easl_source_to_wgsl`, `get_easl_program_info`, `format_easl_source`; also re-exports `pub mod window` when the `window` feature is enabled
- `src/parse.rs` — S-expression parser (uses the `fsexp` crate)
- `src/format.rs` — source formatter
- `src/interpreter.rs` — CPU-side interpreter; also defines the `IOManager` trait, `WindowEvent`/`IOEvent` enums, `GpuBufferKind`, `EvaluationEnvironment`, and `Value::to_uniform_bytes`
- `src/window.rs` — wgpu-based GPU renderer; `GpuCore` struct (public) manages device/surface/pipelines/buffers and is the central GPU resource type; `StdoutIO` opens a real winit window; also exports `create_headless_gpu_core` for surfaceless GPU use
- `src/main.rs` — CLI entry point, currently just runs a compilation benchmark
- `src/compiler/` — the compiler:
  - `core.rs` — top-level compilation entry point
  - `program.rs` — `Program` struct and the main compilation pipeline (`validate_raw_program`). This is the largest and most important file
  - `expression.rs` — `TypedExp` (typed expression tree) and all expression-level transformations (monomorphization, inlining, type inference, etc.)
  - `functions.rs` — `AbstractFunctionSignature`, `FunctionSignature`, monomorphization and higher-order argument inlining for functions
  - `types.rs` — type system: `Type`, `AbstractType`, `TypeState`, `ExpTypeInfo`, type inference, unification, constraints
  - `structs.rs` — `AbstractStruct`, struct monomorphization
  - `enums.rs` — `AbstractEnum`, enum monomorphization
  - `builtins.rs` — all built-in function/struct/macro definitions
  - `effects.rs` — effect types (fragment-exclusive functions, print, window/spawn-window, etc.)
  - `entry.rs` — shader entry point validation (vertex/fragment/compute)
  - `error.rs` — `CompileErrorKind` enum and error reporting
  - `vars.rs` — top-level variables and address spaces
  - `wgsl.rs` — WGSL code generation (final output)
  - `annotation.rs` — `@annotation` parsing
  - `macros.rs` — macro expansion
  - `info.rs` — program info extraction
  - `util.rs` — utilities

## Compilation Pipeline

The main pipeline lives in `Program::validate_raw_program` (program.rs). The major phases, in order:

1. **Name validation** — checks for reserved/invalid names
2. **Mutable arg wrapping** — wraps `@var` function args
3. **Deshadowing** — renames shadowed local bindings to unique names
4. **Type inference** (`fully_infer_types`) — bidirectional type inference with unification
5. **Control flow validation** — checks for expressions after `break`/`return`, validates match exhaustiveness
6. **Associative expansion** — expands `(+ a b c)` into `(+ (+ a b) c)`
7. **Deexpressionification** — lifts expression-position let/match/if blocks into statements
8. **Monomorphization** — replaces generic functions/structs/enums with concrete versions
9. **Inner function extraction** — extracts closures/lambdas as top-level functions
10. **Overloaded function separation** — renames overloaded functions with type suffixes
11. **Higher-order argument inlining** — specializes HoF calls by inlining the function argument
12. **Entry point & effect validation** — checks shader stage constraints
13. **Ownership validation** — checks reference mutability rules
14. **Reference address space monomorphization** — final pass

## Key Concepts

### Naming conventions
- Easl uses **kebab-case** for functions/variables (`make-two-of`) which compiles to **snake_case** in WGSL (`make_two_of`)
- Type names use **PascalCase** (`TwoOf`, `Option`)
- Monomorphized names get type suffixes: `map` with `T=f32` becomes `map_f32`

### Generics & Monomorphization
- Generic functions, structs, and enums use type parameters: `(defn (map T U) [...])`
- Monomorphization creates concrete copies: `map_f32_TwoOf_f32`
- The monomorphizer lives in `TypedExp::monomorphize` (expression.rs) and `AbstractFunctionSignature::generate_monomorphized` (functions.rs)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ella-Hoeppner/easl](https://github.com/Ella-Hoeppner/easl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-12 -->
