---
trigger: always_on
description: Lean 4 library for polynomial functors, interaction trees, and a generic
---

# PolyFun — AI Agent Guide

Lean 4 library for polynomial functors, interaction trees, and a generic
interaction framework over a polynomial substrate. Built on Mathlib.

## Fast Start

1. Run `lake exe cache get && lake build`.
2. To gauge the Polynomial-functor / FreeM substrate, start with
   [`PolyFun/PFunctor/Basic.lean`](PolyFun/PFunctor/Basic.lean) and
   [`PolyFun/PFunctor/Free/Basic.lean`](PolyFun/PFunctor/Free/Basic.lean).
3. To gauge interaction trees, start with
   [`PolyFun/ITree/Basic.lean`](PolyFun/ITree/Basic.lean).
4. To gauge the protocol-flavored interaction framework, start with
   [`PolyFun/Interaction/Basic/TypeTree.lean`](PolyFun/Interaction/Basic/TypeTree.lean)
   and [`PolyFun/Interaction/Basic/Decoration.lean`](PolyFun/Interaction/Basic/Decoration.lean).

`AGENTS.md` is the canonical guide. `CLAUDE.md` is a symlink to this file.

## What This Project Is

PolyFun packages three layers of generic, domain-agnostic infrastructure
that emerged from the cryptographic-protocols formalization in
[`Verified-zkEVM/VCVio`](https://github.com/Verified-zkEVM/VCVio):

1. **Polynomial functors and lenses.** `PFunctor` cores (positions /
   directions), polynomial charts, lenses (Cartesian, state),
   equivalences, free monad `FreeM`, displayed `FreeM`, and the
   `Cofree` / M-type companion. The Spivak-Niu *Poly* category and its
   internal-language fragments live here. The free monads are re-exported
   from upstream [`leanprover/cslib`](https://github.com/leanprover/cslib)
   (a pinned lake dependency): `PFunctor.FreeM` from
   `Cslib.Foundations.Data.PFunctor.Free` and the functor-generic
   `Cslib.FreeM` from `Cslib.Foundations.Control.Monad.Free`; PolyFun adds
   its own API on top.
2. **Interaction trees** in the style of Xia-Zakowski-He-Hur-Malecha-
   Pierce-Zdancewic (POPL 2020), modeled as the M-type of a one-step
   polynomial functor, with strong / weak bisimulation, simulation,
   handlers, and event signatures.
3. **Generic interaction framework** for sequential, two-party,
   multi-party, and concurrent interaction over a `TypeTree` polynomial
   substrate (`TypeTree := PFunctor.FreeM TypeTree.basePFunctor PUnit`), with
   structural decoration, syntax / strategy / execution lenses, and an
   open-process layer for compositional reasoning. Hancock-Setzer
   recursion over interaction interfaces.

PolyFun is intentionally *not* the place for cryptographic content.
Probabilistic semantics, evaluation distributions, oracle-simulation
security definitions, scheme-specific algebra, and concrete-protocol
runtime layers all live in
[`Verified-zkEVM/VCVio`](https://github.com/Verified-zkEVM/VCVio)
and depend on this library.

## Repo Map

- `PolyFun/PFunctor/`: polynomial functors, charts, lenses, equivalences,
  M-type / cofree, free monad and displayed-free machinery.
- `PolyFun/IPFunctor/`: state-indexed polynomial functors (`IPFunctor I`)
  and their indexed free monads. `Free/Basic.lean` holds the single-index
  `FreeM` (state-polymorphic continuations); `Free/Indexed.lean` holds the
  two-index `FreeM₂` carrying a `LawfulIndexedMonad` instance.
- `PolyFun/ITree/`: coinductive interaction trees, same-signature weak
  bisimulation, cross-signature relational trees (`Bisim/CrossSignature.lean`),
  simulation, handlers, event signatures, and finite observation traces.
- `PolyFun/Interaction/`: protocol-flavored generic interaction framework.
  - `Basic/`: `TypeTree`, node contexts, decorations, syntax / shape /
    interaction, strategies, append / replicate / state-chain
    composition.
  - `TwoParty/`: sender / receiver roles, paired strategies, refinement,
    swap, composition.
  - `Multiparty/`: native multiparty local views and per-party profiles.
  - `Concurrent/`: structural concurrent specs, frontiers, processes,
    machines, traces, fairness, liveness, refinement, bisimulation,
    interleaving, observation.
  - `UC/`: open-process / open-theory layer, structural composition
    (interfaces, par, wire, plug), corruption models, environment
    actions, leakage. *Generic only* — security-flavored UC layers
    (computational equivalence, asymptotic security) live in VCVio.
- `PolyFun/Control/`: monad and comonad infrastructure transitively
  required by the above (coalgebra, comonad, free / freecont monad
  algebra, monad iter / hom, lawful re-exports).
- `PolyFun/Control/LTS/Trace.lean`: generic finite visible traces over the
  silent/visible `Control.LTS` layer and preservation by weak simulation.
- `PolyFun/Logic/`: small logic helpers (`HEq`).
- `PolyFunTest/`: separate test / worked-example library (glob
  `PolyFunTest.+`), built by `lake test` and kept out of the `lake lint`
  scope. Holds the dynamical / interaction worked examples and the
  `IPFunctor` `do`-notation smoke tests. Imports `PolyFun` one-way; nothing
  in `PolyFun` depends on it.

## Module Layering

Imports flow strictly downward; cycles are a build error. The canonical
dependency map lives in [`docs/wiki/repo-map.md`](docs/wiki/repo-map.md#conceptual-layering).
Update that map when a change adds a module or changes an import boundary.

New files must respect the documented DAG. Re-exports through
`PolyFun.lean` are auto-generated; do not hand-edit.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Verified-zkEVM/PolyFun](https://github.com/Verified-zkEVM/PolyFun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
