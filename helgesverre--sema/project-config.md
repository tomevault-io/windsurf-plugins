---
trigger: always_on
description: - Build: `cargo build` | Lint: `make lint` (fmt-check + clippy -D warnings) | All tests: `cargo test`
---

# AGENTS.md — Sema (Lisp with LLM primitives, in Rust)

## Build & Test

- Build: `cargo build` | Lint: `make lint` (fmt-check + clippy -D warnings) | All tests: `cargo test`
- Single crate: `cargo test -p sema-reader` | Single test: `cargo test -p sema --test integration_test -- test_name`
- Run file: `cargo run -- examples/hello.sema` | REPL: `cargo run` | Eval: `cargo run -- -e "(+ 1 2)"`
- Integration tests: `crates/sema/tests/integration_test.rs`. Reader unit tests: `crates/sema-reader/src/reader.rs`.

## Architecture (Cargo workspace, 11 crates)

- **sema-core** → NaN-boxed `Value(u64)` struct, `Env` (Rc+RefCell+hashbrown::HashMap), `SemaError` (thiserror), eval/call callbacks (`set_eval_callback`/`set_call_callback`), thread-local VFS
- **sema-reader** → Lexer + s-expression parser → `Value` AST. Handles regex literals (`#"..."`), f-strings (`f"...${expr}..."`), short lambdas (`#(...)`), shebang lines
- **sema-eval** → Trampoline-based TCO evaluator, special forms, module system (`EvalContext` holds module cache, call stack, spans), `call_value` for stdlib callback dispatch, destructuring/pattern matching (`destructure.rs`), prelude macros (`->`, `->>`, `as->`, `some->`, `when-let`, `if-let`)
- **sema-vm** → Bytecode compiler (lowering → optimization → resolution → compilation), stack-based VM with intrinsic opcodes, NaN-boxed fast paths, debug hooks for DAP
- **sema-stdlib** → Native functions across many modules registered into `Env`. Higher-order fns (map, filter, fold) call through `sema_core::call_callback` — no mini-eval.
- **sema-llm** → LLM provider trait + Anthropic/OpenAI/Gemini/Ollama clients (tokio `block_on`), dynamic pricing from llm-prices.com with disk cache fallback
- **sema-lsp** → Language Server Protocol implementation (tower-lsp). Single-threaded backend via mpsc channel. Features: completions, hover, go-to-definition, references, rename, semantic tokens, folding ranges, inlay hints, document highlight, code lens (eval), workspace scanning, scope-aware symbol resolution.
- **sema-dap** → Debug Adapter Protocol server. Breakpoints, stepping (in/over/out), stack traces, variable inspection. Communicates with the bytecode VM via debug hooks.
- **sema-fmt** → Code formatter for Sema source files.
- **sema-wasm** → WASM bindings for browser playground
- **sema** → Binary (clap CLI + rustyline REPL) + `sema build` (standalone executables) + `sema compile`/`sema disasm` + `sema lsp` + `sema dap` + `sema fmt` + integration tests
- Dep flow: `sema-core ← sema-reader ← sema-vm ← sema-eval ← sema-stdlib/sema-llm ← sema`. **Critical**: stdlib/llm depend on core, NOT eval. Stdlib calls eval via thread-local callbacks registered by sema-eval.

## Code Style

- Rust 2021, single-threaded (`Rc`, not `Arc`), `hashbrown::HashMap` for `Env` bindings, `BTreeMap` for user-facing sorted maps.
- Errors: `SemaError::eval()` / `::type_error()` / `::arity()` constructors — never raw enum variants. Use `.with_hint()` for actionable user guidance.
- Native fns: `NativeFn` takes `(&EvalContext, &[Value])` → `Result<Value, SemaError>`. Use `NativeFn::simple()` or `NativeFn::with_ctx()`. Special forms return `Trampoline`.
- Sema naming: slash-namespaced (`string/trim`, `file/read`), predicates end `?`, arrows for conversions (`string->symbol`). Legacy Scheme names kept (`string-append`, `substring`).

## Playground

- Hosted at **sema.run** (WASM)
- Examples live as `.sema` files in `playground/examples/<category>/` subdirectories
- `playground/build.mjs` auto-generates `playground/src/examples.js` from those files — **never edit `examples.js` by hand**
- To add a playground example: add the `.sema` file to the appropriate category dir, then run `cd playground && node build.mjs`
- Categories: `getting-started`, `functional`, `data`, `http`, `llm-tools`, `patterns`, `visuals`, `math-crypto`

## Website

- Hosted at **sema-lang.com**, deployed via `cd website && vercel --prod`
- VitePress site, URLs require `.html` suffix: e.g. `https://sema-lang.com/docs/internals/lisp-comparison.html`
- All docs pages are under `/docs/`: `https://sema-lang.com/docs/...`

## Bytecode File Format (.semac)

- Spec: `website/docs/internals/bytecode-format.md` — **this is the single source of truth**
- Serialization/deserialization lives in `crates/sema-vm/src/serialize.rs`
- **Any change to opcodes, Chunk, Function, ExceptionEntry, or UpvalueDesc MUST update both the format spec and the serializer**
- Format: 24-byte header (magic `\x00SEM` + version + flags), then sections (string table, function table, main chunk, optional debug sections)
- Spur remapping: global opcodes use string table indices in the file, remapped to process-local Spurs on load

## Testing — Dual Eval (Tree-walker + VM)

Sema has **two evaluators**: a tree-walking interpreter (`sema-eval`) and a bytecode VM (`sema-vm`). Both must produce identical results for all pure-computation features. **Any new language feature must be tested through both backends.**

- **Dual-eval test file**: `crates/sema/tests/dual_eval_test.rs` — use `dual_eval_tests!` and `dual_eval_error_tests!` macros
- **Shared harness**: `crates/sema/tests/common/mod.rs` — provides `eval_tw()`, `eval_vm()`, `eval_both()`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HelgeSverre/sema](https://github.com/HelgeSverre/sema) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
