---
trigger: always_on
description: Manages an indent_stack to emit INDENT/DEDENT tokens
---

# Aver — AI Context File

## What is this?

Aver is a programming language designed for AI-assisted development. Its bytecode VM is written in Rust. The language prioritises human and machine readability: every function carries an optional prose description, and architectural decisions are first-class citizens expressed as `decision` blocks co-located with the code they describe. This file is the single entry point for any AI resuming work on this project — read it before touching any source file.

## Project philosophy

- Code is a letter to the next reader — who is increasingly an AI
- Every fragment must be self-sufficient (readable without context)
- Intent over implementation: signatures tell the full story
- Decisions are first-class citizens of the codebase

## AI discovery workflow

When entering this repo, do not start by reading raw source files exhaustively.

Prefer progressive discovery:
- start with `aver context <entry> --budget 10kb`
- use `--focus <fn_name>` to zoom into a specific function's dependency cone
- use the exported architecture map to choose the next module
- raise the budget or target a specific module only when the first map is too shallow

`aver context` is a navigation primitive for AI: start high, focus on what matters, then zoom in.

## Current status

### Language features

See [README.md](README.md) for project overview and workflow, [docs/language.md](docs/language.md) for the surface-language guide, and [docs/services.md](docs/services.md) for the standard library and effectful services.

Below: implementation details relevant to development only.

### Implementation notes

- **Constructor routing**: `Result.Ok(v)`, `Result.Err(v)`, `Option.Some(v)` route through `call_builtin` (`__ctor:Result.Ok` etc.). `Result` and `Option` registered as `Value::Namespace`. Match patterns use qualified names.
- **No flat builtins** (decision: `FullNamespaceEverywhere`). Namespace helpers live under their owning modules (`List`, `Vector`, `Map`, `String`, etc.), with pure list operations in `src/types/list.rs` and vector operations in `src/types/vector.rs`.
- **`Type::Named(String)`** in the type system: capitalized identifiers (including dotted names like `Tcp.Connection`) in type annotations resolve to named types. Compatible only with the same name or internal `Unknown` fallback.
- **`Tcp.Connection` record**: fields `id: String`, `host: String`, `port: Int`. No longer opaque — constructable via `Tcp.Connection(id = ..., host = ..., port = ...)`. Actual socket in thread-local `HashMap` keyed by `id`. `NEXT_ID: AtomicU64` generates "tcp-1", "tcp-2", etc.
- **Static type checker** (`src/types/checker/`): internal `Type::Unknown` recovery after earlier errors so analysis can continue. Bare `Unknown` does **not** satisfy concrete types in constraints — only nested `Unknown` is tolerated (gradual typing). Match pattern bindings are typed: `Result.Ok(x)` on `Result<Int, String>` gives `x: Int`.
- **Auto-memoization** (`src/call_graph.rs`, `src/types/checker/memo.rs`): call graph built from AST, Tarjan SCC detects recursion. Eligibility: pure + recursive + all params memo-safe (scalars, records/variants of scalars). VM caches per-function results (capped at 4096).
- **TCO** (`src/tco.rs`): transform pass rewrites tail-position `FnCall` → `Expr::TailCall` in recursive SCCs. VM handles tail calls via frame reuse. Pipeline: `parse → tco_transform → typecheck → resolve → compile → execute`.
- **Compile-time variable resolution** (`src/resolver.rs`): `Ident("x")` → `Resolved(slot)` inside FnDef bodies.
- **Bytecode VM** (`src/vm/`): stack VM over `NanValue`, with language-shaped opcodes for lists, records, variants, wrappers, tuple literals/patterns, and tail calls. `src/vm/runtime.rs` is the host/effect bridge; `src/vm/execute.rs` is the core loop; `src/vm/compiler.rs` lowers resolved AST to bytecode.
- **`check` command**: warns when module has no `intent =`, function with effects/Result return has no `?` description, file exceeds 250 lines. `fn main()` is exempt from `?` requirement.
- **Entry-point effect enforcement**: `main`/top-level entry calls enforce declared effects at the entry boundary.
- **Opaque types** (`exposes opaque [T]`): module-level access control for types. An opaque type is visible in signatures (can be passed, returned, stored) but cannot be constructed, have its fields accessed, or be pattern-matched from outside the defining module. Enforced at compile time in the typechecker; `load_module_sigs` registers a dummy sig (type resolves) but omits field types, constructors, and variant info. Parser recognizes `exposes opaque` after the `Exposes` token by checking for `Ident("opaque")`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jasisz/aver](https://github.com/jasisz/aver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
