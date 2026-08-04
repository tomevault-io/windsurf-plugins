---
trigger: always_on
description: Operating manual for coding agents working in `zig-js`. `AGENTS.md` is a symlink
---

# CLAUDE.md

Operating manual for coding agents working in `zig-js`. `AGENTS.md` is a symlink
to this file, so any agent convention that reads either name gets the same rules.

Read this before the first edit. When something here disagrees with what you
find in the tree, the tree wins — fix this file in the same change.

---

## 1. What this repository is

A JavaScript engine written from scratch in Zig, with no bundled C engine. It is
three products in one tree:

| Product | Entry point | Notes |
| --- | --- | --- |
| Embeddable Zig engine | [`src/root.zig`](src/root.zig) — `js.Context` | The primary API. |
| JavaScriptCore-shaped **public C API** | [`src/c_api.zig`](src/c_api.zig) | Installs `libzig-js.a` + `JavaScriptCore/`-compatible headers. A pinned *subset target*, never "all of JSC". |
| Versioned **private ABI profiles** | [`src/private_abi.zig`](src/private_abi.zig), [`docs/abi/README.md`](docs/abi/README.md) | Exact, revision-pinned contracts for named downstream consumers (Home, Bun). |

Plus a macOS Objective-C bridge (`src/objc_bridge.m`), a pure-Zig WebAssembly
runtime (`src/wasm/`), and GIL-free shared-realm threading (`src/jsthread.zig`,
`src/gil.zig`).

Dependencies are two sibling Zig packages — `zig-regex` and `zig-gc` — resolved
**by local path**. System JavaScriptCore is used only by explicitly-named
differential and benchmark targets, never by the library itself.

---

## 2. Prerequisites (get this wrong and nothing builds)

- **Zig `0.17.0-dev`**, at least `0.17.0-dev.956`. Zig 0.16 will not build this
  tree. On this machine zig is not on the default `PATH`; the pantry shim is at
  `$HOME/.local/share/pantry/global/bin`.
- **Sibling checkouts** next to this repo — `../zig-regex` and `../zig-gc` (see
  [`build.zig.zon`](build.zig.zon)). Without them dependency resolution fails
  before compilation starts. In a `.claude/worktrees/<name>` worktree you must
  create those two symlinks yourself.
- **Submodules** for corpora: `test262`, `wasm-spec-wg1`, `wasm-spec-wg3`. A
  missing corpus is skipped cleanly rather than failing, which means a run can
  silently score **zero** files — always sanity-check the denominator.
- **`python3`** for the `tools/*.py` gates, **`bun`** for the docs site.

---

## 3. Repository map

```
src/                 the engine
  lexer.zig parser.zig ast.zig      source -> AST
  interpreter.zig                   tree-walking evaluator (the semantic baseline, ~50k lines)
  compiler.zig bytecode.zig vm.zig  AST -> bytecode -> stack VM (suspend/resume, deep recursion, tail calls)
  jit.zig jit/                      baseline native tier + optimizing tier (aarch64)
  value.zig value_nb.zig nanbox.zig NaN-boxed Value, Object, coercions
  shape.zig                         hidden classes / transition tree
  context.zig                       the engine instance: allocators, globals, GC wiring, options
  builtins.zig promise.zig          built-ins and the microtask queue
  gc.zig gc_runtime.zig gc_relocation.zig stack_scan.zig root_handshake.zig  precise/parallel GC
  gil.zig jsthread.zig parallel_lock.zig worker.zig agent.zig  threading
  wasm/                             decode, validate, exec, simd, atomic, gc, JS API
  c_api.zig private_abi.zig private_abi/  embedding ABIs
  *_data.zig cldr_*.zig unicode_*.zig     generated tables (regenerate, never hand-edit)

conformance/         test262 + PR-249 thread corpus + wasm spec runners
tests/               C / C++ / Objective-C embedding fixtures
bench/               benchmark workloads and runners
tools/               python + zig gates, generators, audits, profilers
docs/                the published documentation site (bunpress)
docs/.data/          machine-readable evidence: run inventories, benchmark samples, matrices
reference/           vendored upstream reference material (see the do-not-touch note below)
.github/workflows/   CI — the authoritative gate list
```

`build.zig` is ~95 KB and defines **well over a hundred** named steps. It is the
real index of what this project can do: `zig build --help` (or
`grep -n 'b.step(' build.zig`) beats guessing.

---

## 4. Execution model in one paragraph

The **tree-walker is the semantic baseline** and runs nearly all code. The
bytecode VM exists for *capability*, not speed: generators, async functions, and
async generators need suspend/resume, and the VM's heap activation stack gives
deep recursion and proper tail calls. A plain function only tiers into the VM
when it can actually benefit. Above that sit a baseline native tier and an
optimizing tier, both of which must preserve an exact interpreter fallback.

The practical consequence for agents: **a fix in `interpreter.zig` frequently
needs a mirrored fix in `vm.zig`**, and the reverse. test262 exercises the VM
far less than the tree-walker, so VM/tree-walker divergence is a known bug
surface that the corpus will not catch for you. Use `threadfuzz` and
IIFE/`eval`-shaped probes to hit VM-only paths.

---

## 5. Commands, and what they cost

Always prefer the cheapest command that answers the question.

### Build

```bash
zig build                       # libzig-js.a + headers into zig-out/
zig build test262-bin           # build the corpus runner only (no run)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zig-utils/zig-js](https://github.com/zig-utils/zig-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
