---
trigger: always_on
description: pip install pre-commit && pre-commit install
---

# CVXPY Development Guide

## Quick Reference

### Commands
```bash
# Install in development mode
pip install -e .

# Install pre-commit hooks (required)
pip install pre-commit && pre-commit install

# Run all tests
pytest cvxpy/tests/

# Run specific test
pytest cvxpy/tests/test_atoms.py::TestAtoms::test_norm_inf

# Rebuild the C++ core after changing cvxpy/cvxcore/ (requires swig)
./rebuild_cvxcore.sh
```

Requires Python >= 3.11. Linting is enforced by ruff via pre-commit.

## Code Style

- **Line length**: 100 characters
- **IMPORTANT: IMPORTS AT THE TOP** of files - circular imports are the only exception
- **IMPORTANT:** Add Apache 2.0 license header to all new files

### License Header
```python
"""
Copyright, the CVXPY authors

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
"""
```

## Architecture

### Expression Hierarchy
```
Expression (base)
├── Leaf (terminal nodes)
│   ├── Variable
│   ├── Parameter
│   └── Constant
└── Atom (function applications)
    ├── AffineAtom
    ├── Elementwise
    └── AxisAtom
```

### Reduction Chain
Problems are transformed through a chain of reductions:
```
Problem → [Dgp2Dcp] → [FlipObjective] → Dcp2Cone → CvxAttr2Constr → ConeMatrixStuffing → Solver
```

**Key reductions:**
- `Dgp2Dcp` - Converts DGP (log-log curvature) to DCP (if `gp=True`)
- `Dqcp2Dcp` - Converts DQCP (quasiconvex) to DCP, solved via bisection (if `qcp=True`)
- `FlipObjective` - Converts Maximize to Minimize (negates objective)
- `Dcp2Cone` - Canonicalizes atoms to conic constraints (calls canonicalizers)
- `CvxAttr2Constr` - Converts variable attributes (e.g., `nonneg=True`) to constraints
- `ConeMatrixStuffing` - Extracts A, b, c matrices for solver

Each reduction implements:
- `accepts(problem) → bool` - Can handle this problem?
- `apply(problem) → (new_problem, inverse_data)` - Transform
- `invert(solution, inverse_data) → solution` - Map solution back

See `cvxpy/reductions/solvers/solving_chain.py` for chain construction.

### DCP Rules
Atoms define curvature via:
- `is_atom_convex()` / `is_atom_concave()` - Intrinsic curvature
- `is_incr(idx)` / `is_decr(idx)` - Monotonicity per argument

### DNLP (Disciplined Nonlinear Programming)
DNLP supports smooth nonconvex problems via `prob.solve(nlp=True)`, which bypasses the conic chain and uses `cvxpy/reductions/solvers/nlp_solving_chain.py`. Nonconvex atoms live in the `cp.nlp` namespace (e.g. `cp.nlp.sin`, `cp.nlp.cos`, `cp.nlp.tanh`, `cp.nlp.normcdf`). Requires an NLP solver (IPOPT, UNO, KNITRO, COPT). Check with `problem.is_dnlp()`; transformed via the `dnlp2smooth` reduction.

### DPP (Disciplined Parametrized Programming)
DPP enables efficient re-solving when only `Parameter` values change. CVXPY caches the canonicalization and reuses it.

**How it works**: Parameters are treated as affine (not constant) for curvature analysis. This means:
- `param * param` → NOT DPP (quadratic in params)
- `param * variable` → DPP (affine in params, params only in one factor)
- `cp.norm(param)` in constraint → NOT DPP (nonlinear in params)

Check with `problem.is_dpp()`. See `cvxpy/utilities/scopes.py` for implementation.

## Implementing New Atoms

Copy the structure of an existing atom rather than writing from scratch. The full checklist:

1. **Atom class** in `cvxpy/atoms/` (or `atoms/elementwise/`) - must define `shape_from_args`, `sign_from_args`, `is_atom_convex`/`is_atom_concave`, `is_incr`/`is_decr` (per-argument monotonicity; missing these breaks DCP analysis), and `numeric`
2. **Canonicalizer** in `cvxpy/reductions/dcp2cone/canonicalizers/` with signature
   `def my_atom_canon(expr, args, solver_context: SolverInfo | None = None)` returning `(epigraph_var, constraints)`
3. **Register** it in `CANON_METHODS` in `cvxpy/reductions/dcp2cone/canonicalizers/__init__.py`
4. **Export** the atom in `cvxpy/atoms/__init__.py`
5. **Document** it (see `doc/` folder; renders at [cvxpy.org](https://www.cvxpy.org/))

Steps 3-5 fail silently when forgotten - check them explicitly.

## Testing

Tests should be **comprehensive but concise and focused**. Cover edge cases without unnecessary verbosity. Tests subclass `cvxpy.tests.base_test.BaseTest`, which provides `assertItemsAlmostEqual(a, b, places=5)` for arrays and `assertAlmostEqual` for scalars - copy an existing test file for the pattern.

**IMPORTANT:** Use `solver=cp.CLARABEL` for tests that call `problem.solve()` - it's the default open-source solver.

### Choosing which tests to run

The full suite is slow; run the test files matching your change (e.g. atom change → `test_atoms.py` and `test_constant_atoms.py`; canonicalization → `test_canon_methods.py`; a solver interface → `pytest cvxpy/tests/test_conic_solvers.py -k SOLVERNAME`; NLP features → `tests/nlp_tests/`). Only run broader suites when the change is structural.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cvxpy/cvxpy](https://github.com/cvxpy/cvxpy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
