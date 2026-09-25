---
trigger: always_on
description: Machine-checked cryptographic proofs in Lean, built on Mathlib.
---

# VCVio — AI Agent Guide

Machine-checked cryptographic proofs in Lean, built on Mathlib.

## Fast Start

1. Run `lake exe cache get && lake build`.
2. Read `Examples/OneTimePad/Basic.lean` for a compact modern proof (correctness and privacy).
3. Choose the work area by task: use `VCVio/` for oracle/probability/program-logic work, `LatticeCrypto/` for lattice schemes and reductions, and `LatticeCryptoTest/` for vectors or differential tests.
4. If `𝒟` lemmas fail unexpectedly, check for `[OracleSpec.IsMeasureSpec spec]`
   and the required measurable spaces. The concrete `unifSpec` and `coinSpec`
   have native uniform-measure instances; other specs need a chosen interpretation.

`AGENTS.md` is the canonical guide. `CLAUDE.md` is a symlink to this file.

## Attribution, Headers, And Docstrings

Follow [`CONTRIBUTING.md`](CONTRIBUTING.md) for the repo's explicit attribution policy.

- New Lean files should use the standard copyright / license / authors header and a module docstring.
- For ordinary Lean source files, use the standard prologue layout: header, blank line, `module`, public imports, blank line, module docstring.
- Docstrings must be intrinsic and descriptive. Cross-reference live sibling definitions when helpful, but do not mention removed or renamed declarations, change history, or use reactive wording such as "replaces" or "renamed from".
- Preserve existing headers on routine edits.
- Only rewrite attribution when a file is genuinely new or materially replaced.
- Do not add a separate AI-attribution line.
- For inline section breaks within a Lean file, use Mathlib-style doc-comment headers `/-! ## Title -/` (or the multi-line `/-! ## Title \n\n explanation -/` form). **Do not use ASCII banners** such as `-- ====...===` flanking a `-- § Title` line. The `/-!` form is rendered by `doc-gen4`; ASCII banners are not, and they make the file feel artificially partitioned. If a section is large enough to want a loud header, it is usually large enough to want its own `namespace` or its own file. See *Section Headers Within A File* in [`CONTRIBUTING.md`](CONTRIBUTING.md).

## Module Scopes

Active Lean libraries and tests use Lean's module system. Put declarations in a `public section`;
use `public meta section` for tactic and elaborator code. Existing ordinary source files use
`@[expose] public section` so pre-migration downstream definitional equalities remain available; new
files use plain `public section` and expose individual definitions with `@[expose]` where unfolding
is part of the intended API. The per-library count of broadly exposed files is a ceiling
(`scripts/check-expose-boundary.sh`, baseline `scripts/expose_boundary_baseline.tsv`): converting a
file lowers it, and raising it needs an explicit baseline change under review. Executable and
runtime implementation modules should use opaque `public section` when callers do not need to
unfold their definitions.

- Use `public import` for dependencies that form part of the module's transitive public surface and
  `public meta import` for exported tactic/elaborator dependencies.
- Use plain `import` for implementation-only dependencies. A proof module may use `import all` to
  access an unexposed implementation from the same dependency when proving its public API.
- Do not enable `backward.privateInPublic` or `backward.proofsInPublic`; resolve visibility and
  proof-metavariable issues directly.
- Keep the dormant `Interop` library outside this policy until it is migrated separately.

Section `variable` lines carry an instance assumption only when the theorems in scope share it:
Lean auto-includes an instance-implicit section variable in every theorem mentioning its types,
so a section-wide `[Fintype Chal] [Inhabited Chal] [SampleableType Chal]` that most theorems
ignore turns into an `omit [...] in` line before each of them. Put such assumptions on the
declarations that use them, or open a `section` around the block that shares them; never declare
one that another in scope implies (`[Finite X]` beside `[SampleableType X]`). `omit` is for a
genuine one-off. See *Section Variables* in [`CONTRIBUTING.md`](CONTRIBUTING.md) and gotcha 31.

## What This Project Is

VCVio is a framework for formal cryptographic proofs built around `OracleComp spec α`, the free monad on the polynomial functor induced by an oracle signature `OracleSpec ι := ι → Type`. Its universal fold `simulateQ impl : OracleComp spec α → r α` is the unique monad morphism extending any `impl : QueryImpl spec r` to the free monad. For `OracleComp`, `support` is definitionally `simulateQ` into `SetM` with queries interpreted by `Set.univ`; the primary `evalDist` / `𝒟[…]` semantics is a successful-output Mathlib `Measure`, while `evalSPMF` / `𝒮[…]`, `probOutput`, and `Pr[…]` form the discrete compatibility surface backed by `simulateQ` into `PMF` using `[IsProbabilitySpec spec]`. Uniform cardinality lemmas and the `support`/probability bridge use `[IsUniformSpec spec]`, which bundles `∀ t, Fintype (spec.Range t)`, `∀ t, Inhabited (spec.Range t)`, and uniform sampling. `ProbComp α := OracleComp unifSpec α` specializes to computations whose only oracle is uniform selection.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Verified-zkEVM/VCVio](https://github.com/Verified-zkEVM/VCVio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
