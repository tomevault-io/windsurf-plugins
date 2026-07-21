---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is Malgo?

Malgo is a statically typed functional programming language with an interpreter written in Haskell. Source files use the `.mlg` extension.

## Build, Test, and Development Commands

```bash
mise run setup       # Install toolchain (GHC 9.12.4, cabal, hpack, ormolu)
mise run build       # Format + build (hpack && cabal build)
mise run test        # Run test suite
mise run test -- --match=Parser  # Run tests matching "Parser"
mise run format      # Format with Ormolu
mise run exec        # Run executable (cabal exec malgo-exe)
mise run setup-hls   # Setup HLS for editor integration
mise run reset       # Reset golden test outputs
```

After `cabal install`, use: `malgo eval examples/malgo/Hello.mlg`

## Project Structure

- `src/Malgo/` - Haskell source (modules `Malgo.*`)
- `app/malgo/Main.hs` - CLI entry point (`malgo eval ...`)
- `runtime/malgo/` - Malgo runtime/stdlib (`Builtin.mlg`, `Prelude.mlg`)
- `examples/malgo/` - Sample `.mlg` programs
- `test/Malgo/` - Hspec tests mirroring source structure (`*Spec.hs`)
- `test/testcases/` - Test input files; `.golden/` - golden test outputs

## Compilation Pipeline Architecture

The pipeline is orchestrated in `src/Malgo/Driver.hs`:

```
Source (.mlg)
    ↓
ParserPass → RenamePass → [InferPass] → [RefinePass]
    ↓
ToFunPass → ToCorePass → FlatPass → JoinPass
    ↓
EvalPass (Interpreter) | SchemePass (--target scheme) | ZigPass (--target zig / malgo compile)
```

**Note**: InferPass and RefinePass can be skipped for fast evaluation without type checking.

`ToCorePass` runs `Malgo.Sequent.SaturateCtor.saturateProgram` first thing, before CPS
conversion: it inlines a fully(-or-over-)saturated call of a data constructor
(`Cons x xs`, or `Cons (f x) (mapList f xs)` — arguments need not be
immediate) directly into `Fun.Construct`, instead of invoking the
constructor's own curried closure. This is shared by every backend
(Eval/Scheme/Zig) and every direct caller of `toCore`, not Zig-specific.

### Zig Backend (native executables)

`malgo compile SOURCE [-o OUT] [--opt debug|release-safe|release-fast]` compiles
via Zig to a native executable (Zig 0.16 pinned in `mise.toml`). Pipeline inside
`ZigPass` (`src/Malgo/Backend/Zig/`):

```
Join IR (already saturated — see SaturateCtor above) → Normalize (Mu/Label elimination)
        → ClosureConv.convertProgram (ANF Ir, closure conversion)
        → Peephole (scrutinee-tuple elimination)
        → Perceus (dup/drop insertion) → Reuse (Drop/MkStruct → reuse-token pairing)
        → RcCheck (linearity + reuse-token assert)
        → Emit (Zig text, runtime embedded via file-embed from runtime/zig/runtime.zig)
```

- Memory: Perceus reference counting. Every produced binary leak-checks itself
  at exit (`MALGO-LEAK` on stderr + exit 83 on failure).
- Calling convention is self-passing: `fn(self, args)`; the callee dups its
  captures then drops `self`.
- Allocation-reduction passes (M10/M11): `Peephole` removes the scrutinee
  tuple a multi-parameter clause match otherwise allocates. `Reuse` pairs a
  Perceus `Drop` with a later `MkStruct` in the same block into
  `DropReuse`/`MkStructReuse`, letting the runtime (`rt.dropReuse`/
  `rt.mkStructReuse`) recycle a uniquely-referenced Object in place (Koka-style
  FBIP, generalized to any same-arity payload, not just literal cell reuse).
  Set `MALGO_RC_STATS=1` when running a compiled binary to print
  `MALGO-STATS: total_allocs=<N> reuse_hits=<N>` to stderr.
- Golden parity harness: `bash scripts/zig-golden.sh` (CI job `zig-golden`)
  compiles every golden testcase and diffs stdout byte-for-byte against the
  interpreter's goldens, failing on any leak.
- Runtime unit tests: `zig test -lc runtime/zig/runtime.zig` (`-lc` is required on
  Linux since the runtime calls `std.c.write`/`std.c.getenv` directly; macOS
  masks this because it always links libc via libSystem).
- The interpreter (`Malgo.Sequent.Eval`) is the semantic oracle: any observable
  divergence in the Zig backend is a bug, matched against Eval.hs, not Scheme.

### Intermediate Representations

| IR | Module | Purpose |
|----|--------|---------|
| Fun IR | `Sequent/Fun.hs` | Functional, close to AST |
| Core IR | `Sequent/Core/Full.hs` | Sequent calculus, explicit control |
| Flat IR | `Sequent/Core/Flat.hs` | No nested computations |
| Join IR | `Sequent/Core/Join.hs` | Normalized, explicit join points (final) |

### Key Modules

- `Malgo.Driver` - Pipeline orchestration
- `Malgo.Syntax` - Phase-indexed AST with type families for extensibility
- `Malgo.Pass` - Compiler pass abstraction
- `Malgo.Parser.*` - Parsing (Regular and CStyle variants)
- `Malgo.Rename.*` - Name resolution and desugaring
- `Malgo.Sequent.Eval` - Interpreter for Join IR
- `Malgo.Monad` - Effectful monad stack runner
- `Malgo.Features` - Feature flag system

## Self-Hosting Levels

Malgo has two self-hosting levels, each tested by a CI job:

| Level | Description | Script | CI job |
|-------|-------------|--------|--------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [takoeight0821/malgo](https://github.com/takoeight0821/malgo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
