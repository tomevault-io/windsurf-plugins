---
trigger: always_on
description: Review instructions for the **balance** Python package (weighting and balancing utilities for correcting bias in tabular datasets). Prioritize correctness, statistical validity, reproducibility, and backward compatibility.
---

# Code Review Instructions: `balance` (Python)

Review instructions for the **balance** Python package (weighting and balancing utilities for correcting bias in tabular datasets). Prioritize correctness, statistical validity, reproducibility, and backward compatibility.

For project architecture, build/test commands, and file layout, see `CLAUDE.md` in the repository root (at Meta: `fbcode/core_stats/balance/CLAUDE.md`).

## Review checklist

### 0) PR scope and focus — one idea per PR
- Each PR should represent **one self-contained idea** that is easy to test, review, and accept.
- Flag PRs that combine multiple unrelated changes:
  - mixing feature additions with refactoring
  - addressing multiple independent bugs in one PR
  - combining documentation updates with unrelated code changes
  - bundling multiple distinct features together
- If a PR touches multiple subsystems or modules for unrelated reasons, suggest splitting into focused PRs.
- Large PRs are acceptable if all changes serve a single, cohesive goal (e.g., implementing one feature that naturally spans multiple files).
- When flagging scope issues:
  - list the distinct concerns or ideas present in the PR
  - suggest a logical split (e.g., "Consider separating the refactoring into its own PR")
  - explain how splitting will improve reviewability and testability
- **For externally-authored PRs imported from `facebookresearch/balance`, do not assume open-source review covered API consistency** — run §5.5 explicitly even if the GitHub PR is already approved. CI does not flag taste/consistency drift.

### 1) Correctness and statistical soundness
- Verify the implementation matches the intended method (IPW / CBPS / rake / poststratification).
- Confirm assumptions and constraints are handled explicitly (e.g., positivity, normalization, convergence criteria).
- Check output semantics: shapes, index alignment, column names, and dtype stability.
- Ensure missingness and invalid inputs have well-defined behavior (error vs. warning vs. coercion).
- When the diff touches `balance/interop/diff_diff.py`, verify the
  `weight_type="pweight"` contract is still upheld — diff-diff's
  staggered estimators (CallawaySantAnna, StackedDiD, ImputationDiD,
  HeterogeneousAdoptionDiD, TwoStageDiD, WooldridgeDiD, TROP,
  StaggeredTripleDifference, ChaisemartinDHaultfoeuille,
  TripleDifference, SyntheticDiD) each inline a `weight_type !=
  "pweight"` rejection in their `fit()` method (the canonical example
  is `CallawaySantAnna.fit` in diff-diff's `staggered.py`). The
  history-column drop in `balance.interop._common.drop_history_columns`
  must run before any `dd.aggregate_survey()` handoff to avoid leaking
  `weight_pre_adjust` / `weight_adjusted_*` as covariates. (Symbol-based
  anchors instead of line numbers — line numbers in cross-repo files
  rot silently the moment either repo is touched.)

### 2) Input validation and actionable errors
- Validate early: required columns exist, dtypes are supported, and parameter ranges are enforced.
- Weights: explicitly handle/forbid zero, negative, infinite, or NaN weights (as appropriate for the API).
- Make error messages actionable:
  - name the missing/invalid column
  - list available columns when relevant
  - state how to fix it
- Prefer existing helpers in `utils.py` instead of duplicating validation logic.

### 3) Tests (pytest) — required for behavior changes
- New or changed behavior MUST be covered by deterministic `pytest` tests under `tests/`.
- Exercise edge cases when applicable:
  - missing columns / schema mismatch
  - unexpected dtypes (object/category/int/float/bool)
  - NaN/inf handling in inputs and outputs
  - extreme/boundary weights, clipping, normalization
  - empty dataframes / single-row inputs
- Keep tests stable:
  - avoid order/time dependence and uncontrolled randomness
  - if randomness is necessary, fix seeds and assert with tolerances
- Coverage expectation: aim for >90% coverage on new code (`pytest --cov`).
- Prefer using `from balance import load_data` in tests when appropriate.

### 4) Types and docs (Pyre strict)
- The codebase is Pyre-typed (`# pyre-strict`) with `from __future__ import annotations` on every file.
- New/modified public APIs must have complete type hints. Avoid returning `Any` or widening types unless justified.
- New/modified public functions/classes must include a docstring with at least one concrete usage example.
- MIT license header required on every source file.

### 5) Backward compatibility and deprecations
- Do not silently change defaults, return shapes, column names, or CLI flags.
- If a breaking change is intentional:
  - call it out clearly in the PR summary
  - add migration guidance and “before → after” examples
- For deprecations:
  - use proper warnings
  - document timeline and replacement usage
  - update changelog accordingly

### 5.5) API surface consistency and parameter naming

For **every new public default, parameter name, or string-literal option** the PR introduces, do the following — and quote evidence in your review comment, not just a verdict:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [facebookresearch/balance](https://github.com/facebookresearch/balance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
