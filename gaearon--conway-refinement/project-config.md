---
trigger: always_on
description: `README.md` states the results and reading order. These rules govern changes to the Lean
---

# Conway Refinement working rules

`README.md` states the results and reading order. These rules govern changes to the Lean
development and its generated proof blueprint.

## Mathematical standard

- Lean is the authority for what is proved. State exact quantifiers, ambient hypotheses, zero and
  degenerate cases, inequalities, and equality orientations.
- Use established mathematical terminology in declaration names, statements, docstrings, and the
  blueprint. `blueprint/terminology/FIELD.md`, especially Part 0, records the vocabulary chosen for
  this project.
- Follow Mathlib and dependency spelling in Lean module paths, declaration names, and exact API
  references. Reader-facing prose and the blueprint follow the field map. Thus Lean uses
  `Factorization`, while mathematical prose uses “factorisation”; preserve cited titles verbatim.
- A theorem attributed to a source retains every mathematically relevant source hypothesis. Put a
  stronger generalization in a separate declaration and identify it as such.
- Validate a new definition with characteristic lemmas and a nondegenerate example that separates
  it from the nearest plausible wrong definition. Compilation alone does not establish fidelity.
- Search pinned Mathlib and CombinatorialGames before introducing a definition or substantial
  proof. Reuse their natural namespace, notation, theorem shape, and API when the mathematics
  agrees. Do not edit pinned dependencies in place.
- No declaration under `ConwayRefinement/` may use `sorry`, directly or transitively. The only
  admitted axioms there are `propext`, `Classical.choice`, and `Quot.sound`, as enforced by
  `lake exe axioms`. The root-level Palomar `Challenge.lean` may contain only its advertised hole;
  it imports no project module, is paired with the proved `Solution.lean`, and is checked for exact
  statement drift in CI.

## Repository architecture

Modules are organized by their mathematical objects. General mathematics lives under `Algebra/`,
`Data/`, `FieldTheory/`, `LinearAlgebra/`, `Order/`, `RingTheory/`, `SetTheory/`, and `Topology/`.
The main subject modules live under `HahnSeries/` and `Surreal/`; their subdirectories name the
mathematical construction or conclusion, such as ordinal value, polynomial algebra, factorization,
integer parts, and omnific integers. Source attribution belongs in module documentation and the
source index, not in the directory name.

Imports run from general mathematics to Hahn series to surreal numbers. `Examples/`, `Standalone/`,
and local `Tests/` directories are leaves. `Tests/` contains compiled clients of nearby public APIs,
while `Examples/` contains reader-worthy mathematics.

`lake exe layering` enforces these boundaries. `ConwayRefinement/` is a module root, not a namespace
requirement; generic declarations belong in mathematical namespaces such as `HahnSeries`.

Every Lean file uses the module system. Begin with `module`, use `public import` exactly when an
imported name occurs in an exported type, and put exported declarations in a `public section` or
`public noncomputable section`. Import the module that directly provides each name or instance.
`lake build` is authoritative because the `ConwayRefinement.*` glob compiles every source file.

Keep the pinned Lean, Mathlib, CombinatorialGames, and SubVerso revisions fixed during a
mathematical change. A dependency update is its own reviewed change.

## Standalone entry points

Top-level files in `ConwayRefinement/Standalone/Mathlib/` and
`ConwayRefinement/Standalone/CombinatorialGames/` are mathematical claims intended to be read by a
mathematician as single files.

- A top-level claim file states a substantive result in immediately recognizable mathematical
  language. A reader must not need to follow a project definition to decide what it says; inline
  the relevant predicate or display an exact elementary characterization in the file.
- Keep narration short. Docstrings describe the object or conclusion, never implementation history,
  proof status, or editorial plans.
- A sibling `FooProof.lean` supplies the proof of every closed proposition in `Foo.lean`. The proof
  file should make the connection to the statement easy to inspect and move unrelated machinery to
  `Support/`.
- `Support/` is for definitions and lemmas needed only to elaborate proofs. `Examples/` contains
  results worth reading in their own right.
- Mathlib statement files import only Mathlib. CombinatorialGames statement files may additionally
  import CombinatorialGames. Neither imports another project module. Proof siblings are the explicit
  exception and may import the development.

The compiled isolation and pairing rules are enforced by `lake exe standalone-mathlib`,
`lake exe standalone-combinatorial-games`, and `lake exe proof-links`.

## Lean API and source style

- Keep public definition bodies opaque. Provide the characteristic, membership, coercion,
  evaluation, extensionality, and operation lemmas needed by downstream clients.
- Do not rely on accidental definitional equality across modules. Add a public eliminator or pass
  explicit data. Use `@[expose]` only when computation is intentionally public and a separate client
  demonstrates the need.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gaearon/conway-refinement](https://github.com/gaearon/conway-refinement) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
