---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Tidepool

Compile freer-simple effect stacks into Cranelift-backed state machines drivable from Rust. Haskell expands, Rust collapses. The language boundary is the hylo boundary.

**The core idea — bash++ for LLM agents.** Models are near-natively fluent in
Haskell from decades of training data, the same way they are in bash. Tidepool
presents a "basically GHCi" surface (one-shot `eval` + stateful repl over typed
effects) that inherits that fluency: one eval replaces N tool calls. Two rules
govern all surface work: (1) **the API is the prompt** — mirror canonical
Haskell/GHCi; every deviation is a fluency tax; (2) **the interface evolves as
an optimization loop** — clean-context model usage (wins vs bash, frictions)
drives UX changes and new effects, not speculative design.

---

## Rules

### Locked Decisions

The Key Decisions Reference section below is the source of truth for all architectural decisions. Every entry is final. Do not deviate from locked decisions. Do not re-derive them. If you need a decision that isn't there, escalate to the human.

### Plans

`plans/README.md` tracks the current active plan. Read it before starting new work.

> The agent-swarm orchestration protocol (roles, spawn tools, branch hierarchy)
> is **not** here — it lives in the devswarm role context
> (`~/.exo/roles/devswarm/context/root.md`), loaded each session. This file is
> codebase truth; that file is process truth.

---

## Project Structure

```
tidepool/
├── tidepool/              ← Facade crate + MCP server binary (`cargo install tidepool`)
├── tidepool-repr/         ← Core IR types: CoreExpr, DataConTable, CBOR serial  [CLAUDE.md]
├── tidepool-eval/         ← Tree-walking interpreter (oracle): Value, Env, lazy eval  [CLAUDE.md]
├── tidepool-heap/         ← Manual heap + copying GC for JIT runtime
├── tidepool-bignum/       ← Native ghc-bignum shims (Integer arith without GMP)
├── tidepool-optimize/     ← Optimization passes: beta, DCE, inline, case reduce
├── tidepool-bridge/       ← FromCore/ToCore traits + derive macros
├── tidepool-bridge-derive/← Proc-macro for bridge derives
├── tidepool-bridge-effects/← Single-source bridged-record types shared by handlers + test mocks
├── tidepool-macro/        ← Proc-macros embedding Haskell source as CBOR at build time (haskell_eval!/haskell_inline!)
├── tidepool-effect/       ← Effect handling: DispatchEffect, EffectHandler, HList
├── tidepool-codegen/      ← Cranelift JIT compiler + effect machine  [CLAUDE.md]
├── tidepool-runtime/      ← High-level API: compile_haskell, compile_and_run, cache
├── tidepool-mcp/          ← MCP server library (generic over effect handlers)  [CLAUDE.md]
├── tidepool-handlers/     ← Central effect-request handler arms (`<Eff>Req` matches)  [CLAUDE.md]
├── tidepool-repl/         ← GHCi-style resident-session MCP server  [CLAUDE.md]
├── tidepool-lsp/          ← LSP client + workspace daemon (call graph, hover, refs)  [CLAUDE.md]
├── tidepool-testing/      ← Test utilities + property-based generators (internal)
├── examples/{guess,tide}/ ← Demos: number-guessing game, REPL
├── haskell/               ← Haskell harness (tidepool-extract) + test suite + stdlib  [CLAUDE.md]
│   └── lib/Tidepool/      ← Haskell stdlib (auto-imported in MCP)
├── flake.nix              ← Dev shell (Rust + GHC 9.12 with fat interfaces)
└── CLAUDE.md              ← YOU ARE HERE
```

**Per-crate `CLAUDE.md` files hold the crate-specific docs** (loaded when you work
in that directory):
- `haskell/CLAUDE.md` — rebuilding the toolchain, regenerating fixtures,
  extract diagnostics, the eval stdlib map + structured Ask/Llm surface, Known
  Limits, adding Prelude functions.
- `tidepool-repr/CLAUDE.md` — the self-rolled flat-vector `RecursiveTree` scheme,
  `DataConTable` hygiene (`insert_checked`, sibling-group disambiguation),
  session-id newtypes, CBOR wire-format versioning.
- `tidepool-eval/CLAUDE.md` — the JIT's differential oracle: trampoline
  join-point evaluation, WHNF-only `Value`, thunk lifecycle, how it's actually
  tested (differential harnesses, not its own unit suite).
- `tidepool-codegen/CLAUDE.md` — JIT/effect/cache diagnostics, case-trap → `emit_case_trap` (poison + breadcrumb, not SIGILL).
- `tidepool-mcp/CLAUDE.md` — eval-authoring patterns (aperture/census/diff verbs),
  structural search, how to add an effect.
- `tidepool-handlers/CLAUDE.md` — the Rust side of the effect contract: adding a
  handler arm, the four `cx.respond*` variants, sandbox enforcement.
- `tidepool-repl/CLAUDE.md` — resident-session block-runner (decl/stmt/meta item
  classification), the single-owned `SessionState` lifecycle machine, ask/suspend
  mechanism, repl-specific usage notes.
- `tidepool-lsp/CLAUDE.md` — the `tidepool-lsp-daemon` sidecar: socket
  resolution, name/path-only protocol design, op-surface boundaries.

The live **eval API reference** (what eval users can call) is the MCP `eval` tool
description emitted by the server — not duplicated in these files (it drifts).

## Build & Test

```bash
nix develop                              # Enter dev shell (provides Rust + GHC 9.12)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tidepool-heavy-industries/tidepool](https://github.com/tidepool-heavy-industries/tidepool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
