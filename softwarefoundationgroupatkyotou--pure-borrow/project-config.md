---
trigger: always_on
description: This file is the source of truth for coding-agent guidance in this repository.
---

# AGENTS.md

This file is the source of truth for coding-agent guidance in this repository.
Keep agent-agnostic project instructions here, and keep tool-specific files as thin adapters.
`CLAUDE.md` is intentionally a symlink to this file so Claude Code and Codex read the same repository policy.

## Overview

**pure-borrow** realizes **Rust-style borrowing in Linear Haskell, purely** — compile-time ownership and memory safety with no runtime overhead, plus safe deterministic parallelism.
It is the artifact of the paper *Pure Borrow: Linear Haskell Meets Rust-Style Borrowing* (Y. Matsushita & H. Ishii, PLDI 2026; [arXiv:2604.15290](https://arxiv.org/abs/2604.15290)).
The package is already released on Hackage (version `0.0.0.0`); current work is **incremental improvement** (notably performance) of a published, paper-backed library — so preserve the public API and the soundness invariants unless a change is deliberate.

`Control.Monad.Borrow.Pure` is the umbrella module and carries the full Haddock tutorial; read it before designing changes to the core.

## Language & toolchain

- **GHC 9.10.2+** required (Linear Types); **9.12.4+ recommended** — pinned locally to `ghc-9.12.4` via `cabal.project.local`.
  `tested-with: 9.10.3 || 9.12.4 || 9.14.1`.
- `default-language: GHC2021`, with `LinearTypes` enabled by default everywhere.
- **Interpreter caveat:** GHC < 9.12.3 segfaults evaluating some linear programs in GHCi / the HLS eval plugin (a compiler bug).
  Compiled code is fine on 9.10.2+.
  Use 9.12.3+ if you need the REPL.
  The `pure-borrow-doctests` suite is disabled for GHC < 9.12.3.
- Cabal **nix-style builds only**; `cabal.project` (+ `cabal.project.local`) is the source of truth.
  Do not use stack or invoke `ghc` directly.
  cabal-install ≥ 3.14.2 recommended.

## Build / test / bench / run

`cabal.project` sets the `+examples` flag and `cabal.project.local` enables tests+benchmarks, so locally every component below is buildable.

```bash
cabal build all                       # build the library + all enabled components
cabal build pure-borrow               # just the library

cabal test                            # run all test suites
cabal test pure-borrow-test           # main tasty suite only
cabal test pure-borrow-doctests       # doctests (needs GHC >= 9.12.3)

cabal bench qsort-bench               # parallel quicksort benchmark (tasty-bench)
cabal bench fft-bench                 # parallel FFT benchmark (tasty-bench)
cabal bench pure-borrow-bench         # all single-threaded micro-benchmarks (needs -j1)

cabal run qsort -- --help             # quicksort demo executable (needs +examples)
cabal run fft   -- --help             # FFT demo executable (needs +examples)
```

Prefer HLS (via the haskell skill) for iterating; it is far faster than a full `cabal build` for typecheck / hover / go-to-definition / find-references / rename.

#### Running the suites is mandatory

Any change under `src/`, `test/`, `bench/` or `internal-src/` must be validated with a full `cabal build all` followed by `cabal test` **before it is committed**.
Not "when it looks risky" — every time.
A change that only moves code between modules still changes what GHC optimizes and what the Core inspections see.

Run the plain `cabal build all` first, and separately.
`pure-borrow-doctests` extracts its examples from the built library, so `cabal test` on its own can silently exercise a stale build.

#### Build and test at `-O2`

`cabal.project` pins `optimization: 2`, matching the `--enable-optimisation=2` that CI configures with.
Do not work around it with `-O1` or `--disable-optimisation` to save time.
`pure-borrow-inspection` asserts properties of *optimized* Core — that a loop carries no type-class dictionary, that a generic vector specializes.
At `-O1` those assertions hold whether or not the library actually specializes, so a green run at the default level says nothing about whether CI will pass.
This is not hypothetical: a merge that kept every suite green locally at `-O1` failed every GHC in CI on exactly this assertion.

### Tests — tasty + tasty-discover (property tests via `falsify`)

`test/Main.hs` is only the `tasty-discover` driver (`-optF --tree-display`); real tests live in `test/**/*Spec.hs`, in modules mirroring the source tree.
Filter with tasty's `-p`:

```bash
cabal test pure-borrow-test --test-options='-p "Lifetime"'   # run a subset by pattern
```

`test/Control/Monad/Borrow/Pure/Lifetime/TypingCases.hs` holds type-level (compile-time) constraint checks, not runtime assertions.

Two kinds of failing test look superficially alike here, and they encode opposite intentions.
Never convert one into the other.

**"This must not typecheck."** Rejection is the specification, and a case that started compiling would be a soundness bug.
Define the intentionally ill-typed cases in a separate `TypingCases` module compiled with `-fdefer-type-errors -Wno-deferred-type-errors`.
In the corresponding `*Spec.hs`, force each case to WHNF with `evaluate`, catch the exception with `try`, and inspect its diagnostic text.
Do not use `expectFail`, `expectFailBecause`, or another expected-failure wrapper: the test itself must pass only when it observes the intended deferred type error.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SoftwareFoundationGroupAtKyotoU/pure-borrow](https://github.com/SoftwareFoundationGroupAtKyotoU/pure-borrow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
