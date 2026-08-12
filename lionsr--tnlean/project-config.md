---
trigger: always_on
description: This file provides guidance to AI coding assistants working with code in this repository.
---

# CLAUDE.md

This file provides guidance to AI coding assistants working with code in this repository.

## Project Overview

TNLean is a Lean 4 formalization of the **Fundamental Theorem of Matrix Product States**, **Quantum Wielandt theory**, and finite-dimensional **quantum-channel theory** (following Wolf's *Quantum Channels & Operations*). Built on Mathlib v4.32.0.

## Build Commands and Mathlib Cache Policy

**Canonical cache rule:** never rebuild Mathlib from source in a fresh, cloned,
or cache-cleared worktree. After adding or updating a Mathlib dependency, fetch
its prebuilt artifacts **before** any `lake build` or local Lean check:

```bash
# Fetch pre-built Mathlib artifacts after a Mathlib/toolchain/dependency update.
# Do this before `lake build` or `lake env lean`; otherwise Mathlib can rebuild
# from source and take hours.
lake exe cache get

# Only after the cache fetch succeeds:
lake build
lake env lean TNLean/Path/To/File.lean
```
# Check for sorrys/axioms in changed files
rg -n "sorry|axiom" TNLean/Path/To/File.lean || true

# Blueprint validation (requires leanblueprint; run after lake build succeeds)
cd blueprint && leanblueprint checkdecls

# Blueprint web/PDF generation
cd blueprint && leanblueprint web
cd blueprint && leanblueprint pdf
```

## Lean Toolchain & Dependencies

- **Lean**: v4.32.0 (pinned in `lean-toolchain`)
- **Mathlib**: v4.32.0
- **checkdecls**: Blueprint declaration checker (PatrickMassot/checkdecls)
- **Gametheory**: Custom Brouwer fixed-point theorem library (LionSR/Brouwer)

### Lean Options (lakefile.toml)

- `relaxedAutoImplicit = false` — strict implicit arguments, no auto-implicit
- `pp.unicode.fun = true` — pretty-prints `fun a ↦ b`
- `maxSynthPendingDepth = 3` — typeclass synthesis depth limit

## Architecture

The source lives in `TNLean/` and is organized into **layers 0-6 with sublayers**.
See `docs/import_structure.md`; `TNLean.lean` is generated.

| Layer | Modules | Content |
|-------|---------|---------|
| **0** | `Algebra/`, `Analysis/`, `Topology/`, `Axioms/` | Matrix lemmas, trace pairings, Gram matrices, Frobenius norms, Skolem-Noether, cocycle cohomology, Brouwer FPT |
| **1-2** | `Channel/` (Basic, Choi, Kraus, Stinespring, Transfer) | Quantum channel representations (Wolf Ch. 2) |
| **2b-2c** | `Channel/Schwarz/`, `Channel/FixedPoint/`, `Channel/Irreducible/`, `Channel/Peripheral/`, `Channel/Semigroup/`, `Channel/KoashiImoto/`, `QPF/`, `Spectral/` | Kadison-Schwarz, Perron-Frobenius, spectral theory, peripheral spectrum, GKSL semigroups (Wolf Ch. 5-7); common invariant algebra of jointly invariant states (HJPW appendix) |
| **3** | `MPS/Defs`, `MPS/Chain/`, `MPS/Core/`, `MPS/Overlap/` | MPSTensor definition, word evaluation, blocking, transfer matrices, overlap matrices |
| **4** | `MPS/FundamentalTheorem/`, `MPS/Symmetry/` | Single-block FT, gauge equivalence, on-site/virtual symmetries, cocycle coboundary |
| **5** | `MPS/BNT/`, `MPS/CanonicalForm/`, `MPS/Structure/`, `MPS/Irreducible/`, `MPS/Periodic/`, `MPS/FundamentalTheorem/Multi/` | Multi-block assembly, BNT canonical forms, permutation rigidity, periodic tensors |
| **5b** | `MPS/RFP/` | Renormalization fixed-point scaffolding |
| **6** | `Wielandt/` | Span-growth, rank-one extraction, rectangular span, Wielandt bound, primitivity equivalences |

**Other modules**: `PiAlgebra/` (pi-algebra FT variants), `PEPS/` (two-dimensional fundamental-theorem development for torus, cycle, and normal-tensor routes), `MPS/MPDO/` (density operator foundations), and `Archive/` (legacy, excluded from root imports).

### Key Types and Definitions

- `MPSTensor d D` — a `Fin d`-indexed family of `D*D` complex matrices
- `evalWord A w` — product of matrices along word `w : List (Fin d)`
- `IsInjective A` — matrices of `A` span the full matrix algebra
- `SameMPV A B` / `SameMPV₂` — same matrix product vector family
- `GaugeEquiv A B` — conjugation by invertible matrix (`B i = X * A i * X⁻¹`)
- `IsBNTCanonicalForm` — paper-faithful basis-of-normal-tensors canonical form predicate
- `cumulativeSpan A n` — span of all products of length <= n
- `IsNormal A` — the project's normality notion for Wielandt theory
- `transferMap A` — the CP map `rho -> sum_i A_i * rho * (A_i)^H`

## Conventions & Style Guides

Detailed conventions live in `docs/`. Read the relevant file before working in that area:

| File | Covers |
|------|--------|
| [`docs/MATHLIB_style.md`](docs/MATHLIB_style.md) | Code formatting, line length (100 chars), declarations, tactic style, whitespace, transparency, deprecation |
| [`docs/MATHLIB_naming.md`](docs/MATHLIB_naming.md) | Capitalization rules, symbol-to-name dictionary, variable conventions |
| [`docs/MATHLIB_doc.md`](docs/MATHLIB_doc.md) | Module docstrings, definition docstrings, sectioning comments, BibTeX citations |
| [`docs/CONTRIBUTING.md`](docs/CONTRIBUTING.md) | PR title format (`type(scope): description`), issue conventions, label taxonomy, review checklist, mathematical-language renames |
| [`docs/glossary.md`](docs/glossary.md) | Canonical public predicates, mathematical meanings, source anchors, sanctioned bridges, and caveats |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LionSR/TNLean](https://github.com/LionSR/TNLean) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
