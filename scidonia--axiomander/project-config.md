---
trigger: always_on
description: provides goal inspection (`coq_open_goals`), speculative tactic execution
---

# AGENTS.md

## Project

Axiomander is a Hoare-logic verification pipeline for Python.

Users annotate Python functions with `@requires`, `@ensures`, `@invariant` decorators.
The pipeline translates these into weakest-precondition proof obligations in Coq.
SMT solvers clear the easy goals; an LLM oracle generates proofs for the rest.

## Design Philosophy

Axiomander aims to be **the gold standard for verification systems**.
Every architectural decision must be *the right way* from the ground up.
No half-baked approaches. No shortcuts that paper over a fundamental mismatch.
If a feature requires two representations, unify them. If a type system is
incomplete, extend it fully rather than encoding around the gaps.
The goal is not to pass tests — it's to build a sound, composable,
and extensible verification stack that holds up under real-world use.

**Ground rules:**
- Python's runtime type system must be reflected in the `value` type.
  Coercion rules must follow Python's semantics (float+int→float, etc.).
- Contracts are plain Python — `assert` statements in the function body (zero imports,
  zero decorators) or an `axiomander:` docstring block with `requires:`/`ensures:`.
  The assertion_finder classifies `assert` by position; the docstring parser
  converts Nagini-style contracts into the same IR.
- **Contracts are never weakened to pass a test.** If a prover can't handle a
  contract, the fix is in the prover — not in the contract. A failing test with
  an honest `xfail` documents the gap; a "passing" test with a watered-down
  contract hides it. The contract is the specification; the prover serves it,
  not the other way around.
- The WP calculus is the single source of truth, proven sound in Coq.
- Frame conditions are explicit, enforced, and derive from the callee's
  declared `reads`/`writes`, not from implementation details.
- `simpl`/`cbn` reduction is controlled — Fixpoint reduction must not
  expand structural comparisons (`In`, `clobber`) before lemmas fire.
- Immutable values (VList, VTuple, VDict) are structural, like Dafny's
  `seq` and F*'s `list`. Mutable operations (append, pop) work on a
  separate heap representation that parallels the value.
- Every new type or operation must have negative tests that demonstrate
  the verifier catches violations, not just passes trivially. Negative
  tests prove the system isn't "proving everything."

### Open Architecture Decisions

**Per-Callee Frame Lemmas — IN PROGRESS.** CCall frame conditions produce
a `forall x, ~ In x (target :: writes) -> ...` subgoal that causes WP term
blowup and compiled-Ltac pattern matching problems.  Solution: generate one
Coq lemma per (callee, frame-variable) pair at the Python IR level.  Each
lemma is trivial (`apply wp_ccall_frame`).  The caller proof uses `apply
lemma_name` instead of matching the general forall.  Design at
[`docs/frame-lemmas.md`](docs/frame-lemmas.md).

**Lemmas generate correctly;** remaining work is wiring them into the proof
so each `apply lemma_name` fires independently, producing residual state on
failure.

### Staged Proof Engineering — Assessment

A gap analysis against [`staged_proof_engineering_guide.md`](staged_proof_engineering_guide.md):

| Principle | Current | Gap |
|-----------|---------|-----|
| Small named obligations | Per-function theorems + VCG lemmas | CCall frames are one `forall`; per-variable lemmas being built |
| Durable artifacts | Binary cache per function | No tactic trace, no residual goals, no per-obligation dirs |
| Mechanical tactic ladder | `wp_reduce` → `wp_prove` | No trace of which tactic fired/succeeded/failed |
| Residual goal capture | Raw `coqc` error string | No structured residual state with hypotheses |
| Narrow LLM tasks | Prompt gives context | Doesn't include residual goal with hypotheses |
| Multi-level caching | Function-level hash | No per-obligation, per-stage caching |
| Failure classification | `UNPROVED/COUNTEREXAMPLE/LEVEL*` | Doesn't distinguish "bug" vs "missing lemma" vs "weak automation" |

**Root cause:** `wp_prove.` is a single monolithic tactic. When it fails we
lose all intermediate proof state. The fix is to break proofs into staged
named obligations that can fail independently.

### Next Steps (priority order)

1. **Wire frame lemmas into proofs** — replace the general `forall` frame
   subgoal with `apply inc_frame_a. apply inc_frame_b. ...` per variable.
   Each `apply` can fail independently with its own residual state.

2. **Staged proof output** — generate proofs as sequences of named stages
   (preconditions, frame lemmas, postcondition) instead of `wp_prove.` only.

3. **Residual goal capture** — when a stage fails, save the goal state
   with hypotheses to a `.v` fragment for coq-lsp / LLM consumption.

4. **Tactic trace** — log which tactic fired, which subgoal it produced,
   and whether it succeeded or failed. Feed into caching.

5. **Failure classification** — distinguish "weak invariant" from
   "missing lemma" from "arithmetic too complex for lia/SMT."

**Loop predicates — RESOLVED.** Predicates containing loops are handled via
postcondition-inlining: the predicate is verified as a standalone function, its
semantic postcondition (guarded by `implies(result == 1, property)`) is extracted,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scidonia/axiomander](https://github.com/scidonia/axiomander) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
