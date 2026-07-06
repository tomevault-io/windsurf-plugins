---
trigger: always_on
description: Canonical instructions for any coding agent working in this repo. `CLAUDE.md` just points here.
---

# AGENTS.md — Sema (Lisp with LLM primitives, in Rust)

Canonical instructions for any coding agent working in this repo. `CLAUDE.md` just points here.

## Build & Test

```bash
make build              # dev build
make release            # optimized build
make test               # all tests (http tests ignored)
make test-http          # HTTP integration tests (requires network)
make lint               # fmt-check + clippy -D warnings
make fmt                # cargo fmt
make install            # install to ~/.cargo/bin (LTO, no PGO)
make install-pgo        # PGO build + install (slower build, faster runtime)
make all                # lint + test + build
make run                # start REPL
make example-notebook   # run demo notebook headlessly
make test-notebook-e2e  # Playwright E2E tests for notebook
```

- Single crate: `cargo test -p sema-reader` | Single test: `cargo test -p sema --test integration_test -- test_name`
- Single eval test: `cargo test -p sema --test eval_test -- test_name` | Ignored tests: `cargo test -p sema -- --ignored`
- Run file: `cargo run -- examples/hello.sema` | REPL: `cargo run` | Eval: `cargo run -- -e "(+ 1 2)"`
- Integration tests: `crates/sema/tests/integration_test.rs`. Eval tests: `crates/sema/tests/eval_test.rs`. Reader unit tests: `crates/sema-reader/src/reader.rs`.
- Editor plugins live in their own repos under the `sema-lisp` org (`vscode-sema`, `zed-sema`, `intellij-sema`, `emacs-sema`, `helix-sema`, `sema.nvim`, `sema.vim`, `sublime-sema`) and the grammar in `sema-lisp/tree-sitter-sema` — they are no longer in this repo. Each carries its own CI/publishing.

## Architecture (Cargo workspace)

Dependency flow (arrows = "depends on"): `sema-core ← sema-reader ← sema-vm ← sema-eval ← sema-stdlib/sema-llm ← sema`. **Critical**: `sema-stdlib` and `sema-llm` depend on `sema-core` but NOT on `sema-eval` (avoids circular deps). Stdlib calls eval via thread-local callbacks registered by sema-eval.

- **sema-core** → NaN-boxed `Value(u64)` struct, `Env` (Rc+RefCell+hashbrown::HashMap), `SemaError` (thiserror), `EvalContext`, eval/call callbacks (`set_eval_callback`/`set_call_callback`), thread-local VFS
- **sema-reader** → Lexer + s-expression parser → `Value` AST. Handles regex literals (`#"..."`), f-strings (`f"...${expr}..."`), short lambdas (`#(...)`), shebang lines
- **sema-vm** → Bytecode compiler (lowering → optimization → resolution → compilation), stack-based VM with intrinsic opcodes, NaN-boxed fast paths, debug hooks for DAP. **The sole evaluator** (the tree-walker has been retired).
- **sema-eval** → `Interpreter`, special forms, macro expansion (VM-native), module system (`EvalContext` holds module cache, call stack, spans), `call_value` for stdlib callback dispatch, destructuring/pattern matching (`destructure.rs`), prelude macros (`->`, `->>`, `as->`, `some->`, `when-let`, `if-let`)
- **sema-stdlib** → Native functions across many modules registered into `Env`. Higher-order fns (map, filter, fold) call through `sema_core::call_callback` — no mini-eval.
- **sema-llm** → LLM provider trait + Anthropic/OpenAI/Gemini/Ollama clients (tokio `block_on`), dynamic pricing from llm-prices.com with disk cache fallback
- **sema-lsp** → Language Server Protocol (tower-lsp). Single-threaded backend via mpsc channel. Completions, hover, go-to-definition, references, rename, semantic tokens, folding ranges, inlay hints, document highlight, code lens (eval), workspace scanning, scope-aware symbol resolution.
- **sema-dap** → Debug Adapter Protocol server. Breakpoints, stepping (in/over/out), stack traces, variable inspection via VM debug hooks.
- **sema-notebook** → Jupyter-inspired `.sema-nb` JSON notebook format, eval engine with shared cell environment, HTTP server + REST API, embedded browser UI, Markdown export
- **sema-mcp** → Model Context Protocol server exposing Sema eval/build/notebook tools to AI agents
- **sema-otel** → OpenTelemetry facade (spans/metrics); native-only, no-op on wasm32
- **sema-workflow** → Dynamic-workflow runtime: journals a frozen JSONL run-directory, bounded concurrency for leaves, `--resume` via memo sidecar. Leaf crate — depends only on sema-core + sema-otel.
- **sema-docs** → Builtin docs index generator. Each builtin is a markdown file in `crates/sema-docs/entries/`; `sema-docs gen` produces a JSON index consumed by LSP hover/completion and REPL apropos.
- **sema-fmt** → Code formatter for Sema source files
- **sema-wasm** → WASM bindings for the browser playground at sema.run
- **sema** → Binary: clap CLI + reedline REPL + `sema build` (standalone executables) + `sema compile`/`sema disasm` + `sema lsp` + `sema dap` + `sema fmt` + `sema notebook` + integration tests. REPL submodules live in `crates/sema/src/repl/` (editor, highlighter, hinter, validator, inspector, commands).

## Key Design Patterns

- **Trampoline TCO** — `eval_step` returns `Trampoline::Value(v)` (done) or `Trampoline::Eval(expr, env)` (tail call). Special forms must return `Trampoline::Eval` for tail positions to enable proper tail-call optimization.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sema-lisp/sema](https://github.com/sema-lisp/sema) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
