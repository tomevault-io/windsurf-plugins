---
trigger: always_on
description: Orientation for an AI agent (or a new contributor) working *in* this repo. The README is the user-facing
---

# CLAUDE.md

Orientation for an AI agent (or a new contributor) working *in* this repo. The README is the user-facing
guide (install, verbs, benchmarks, the modeled subset); this file is the map of the codebase and the rules
for changing it without breaking soundness or the gate. Read this first, and reach for a specific module
only when you touch it — you should not need to read the whole tree.

## What Touchstone is, in one paragraph

A function plus a property in, a verdict out: **PROVED** (holds for all inputs), **REFUTED** (with a
counterexample), or **UNKNOWN** (with a reason). It translates a modeled subset of Python to Z3
(corroborated by cvc5) rather than running it. A core slice — the integer IR and its weakest-precondition
generators, the division/modulo and fixed-width encodings, the interval transfers, the type-lattice join —
is machine-checked in Rocq and run as the *extracted* code; everything else is modeled soundly and
continuously cross-checked against CPython.

## Package map (`touchstone/`)

| module | responsibility |
| --- | --- |
| `core.py` | the symbolic core: the modeled-subset parser/desugar, the value lattice (`_Opaque`/`_FieldVal`/`_SafeContainer`/`_SetExpr`/`_NdArray`/`_DictParam`/`_MapVal`/…), `ev`/`symexec` (the loop-free value engine), set-operation content (`_set_binop`), strided slicing (`_slice_len`), the ord/chr codepoint bijection and bytes element values, opaque-object field modeling (`_FieldVal`), the tensor (numpy/torch) shape algebra (`_nd_method`/`_matmul`/`_torch_func`), the `math`-module and trap-free standard-library models (`_math_call`/`_fp_pow`/`_STDLIB_TF`), the loop-free heap/OO engine (`_heap_eval`, C3 MRO, method/operator dispatch), the recursive-callee contract summary (`Ctx._contract_summary`), the out-of-process sandbox, and the solver layer (`_solve`, `_solve_corro`, `solve_corroborated` confirming a PROVED with cvc5 *sequentially* -- the cvc5 binding holds the GIL through its solve, so a thread beside z3 is no real overlap and only starves the main thread). The runtime flags live here. |
| `engines.py` | the verbs (`check`, `prove`, `verify_equiv`, `verify_contracts`, `verify_change`, `scan`, …) and the deductive/CHC engines (CFG→Horn trap-freedom with per-loop invariants for nested loops, single-loop invariant synthesis, recursion incl. the array-encoded recursive-list engine, the generator-yield engine with bounded accumulator unrolling, the relational for-loop-equivalence product, interprocedural/whole-program CHC, termination, BMC, the None/Optional engines, exact bitvector). Path-sensitive definite assignment (`_use_before_def`) and the recursive-callee contract summary live here too. The module loaders (`load_module`/`load_program`/`verify_repo`). |
| `domains.py` | abstract interpretation (interval, zone, octagon, Karr, template-polyhedra, machine-integer, float-interval; each PROVED re-derived by the CHC engine via `_corroborate_domain`); the specialized theories decided in Z3 / by enumeration (concurrency: bounded interleavings + inductive + rely-guarantee over locks, counting semaphores, condition variables, async/await; IEEE-754 finiteness; separation logic via cvc5; SOS nonnegativity; sequence/string/dict laws); and the broadest trap-freedom decider (`_decide`, a cascade of `_trap_free_*` strategies) with the differential CPython oracle and the random fuzz corpora. |
| `vcgen.py` | the Rocq-extracted weakest-precondition generator (`wpg`/`vcg`), driven through `_generated/vcgen_rocq.py`, with a CPython cross-check of the Python→IR lowering; the re-checkable proof bundles (`proof_bundle`/`recheck_bundle`); and the SMTCoq obligation export (`python -m touchstone.vcgen` regenerates `proofs/touchstone_obligations.v`). |
| `inference.py` | sound (over-approximating) type inference (`infer_types`/`infer_return_type`/`infer_local_types`: a returned type-set provably contains the runtime type, else UNKNOWN) and the heuristic exact inferencer (`emit_facts`, the TypeEvalPy surface). |
| `audit.py` | `run_self_tests` (the assert suite) + the differential/soundness audits + `demo`. |
| `ci.py` | the gate: `python -m touchstone.ci` runs the self-tests, soundness audits, differential regressions, fuzz, and the committed-extraction audits. Must stay green. The bulk CPython-cross-checked phases run with `REQUIRE_CORROBORATION` off (CPython is a stronger oracle than cvc5's second opinion there); self-tests and the verification showcase keep it on, so the corroboration path stays exercised. |
| `cli.py` / `mcp.py` / `lsp.py` | the CLI verbs (one per use mode), the MCP server (verifier tools over stdio), the LSP. |
| `diagnostics.py` / `repro.py` | UNKNOWN classification + the `covers` reference; REFUTED→runnable failing test. (Re-checkable proof bundles live in `vcgen.py`.) |
| `_generated/*_rocq.py` | the Rocq extraction transpiled to Python — **do not hand-edit**; regenerate from `proofs/`. `_impl.py` re-exports the API; `__init__.py` loads lazily (z3 is imported only on first verification use). |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CharlesCNorton/touchstone](https://github.com/CharlesCNorton/touchstone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
