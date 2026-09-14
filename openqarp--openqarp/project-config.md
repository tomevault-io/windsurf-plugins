---
trigger: always_on
description: Ground rules for AI assistants (and a two-minute refresher for humans).
---

# OpenQARP — agent guide

Ground rules for AI assistants (and a two-minute refresher for humans).
The authoritative conventions document is
`docs/contracts/qarp_conventions.md` — **when code and that
doc disagree, the doc wins**. Read §13 (blocks), §14 (engines/devices),
§15 (repo), §17 (symbols), §18 (test oracles), §19 (resources) before
touching those areas.

## Contribution workflow (plan-first)

- Standard/structural work starts as a plan in `docs/contributions/` (its
  README has the tier table and the loop; `_template.md` is the scaffold).
- **One PR per contribution, two gates inside it.**  Open a *Draft* PR whose
  first commit is the plan; the reviewer green-lights the design there, and
  only then does implementation start — on the same branch, in the same PR.
  Mark the PR ready when the implementation is done; that is the second gate.
- The PR declares deviations from the *green-lit* plan — silent drift is the
  violation.  Fold declared drift back into the plan file before merge.
- Trivial fixes (typo, doc fix, bugfix + regression test) need no plan.
- Plan tooling is two plain-markdown files, readable by any agent —
  `plan/SKILL.md` (scaffold a plan) and `plan-review/SKILL.md`
  (conformance-review a PR against its plan).  Read them directly; they carry
  no tool-specific syntax.  They sit under `.claude/skills/` only because that
  is where Claude Code discovers them, which is also what lets it expose them
  as `/plan` and `/plan-review`.

## Commands

- Editable install: `pip install -e ".[full-dev]"` (`[dev]` is ruff, mypy
  and pre-commit only — no pytest, no optional backends; `[full-dev]` is not
  every extra — `docs`, `notebooks`, `integrations`, `bench` and
  `cudaq-runtime` are excluded).  C++ edits then need
  the install re-run.  Run it **inside the repo-root venv** (`python -m venv
  .venv && source .venv/bin/activate`) — a system Python is PEP 668
  externally-managed and pip refuses to install into it.  One `.venv` per
  checkout, worktrees included: a shared venv pins a single checkout, so the
  others silently test the wrong branch.
- Auto-rebuild-on-import (C++ work only) needs **both** lines, never one:
  `pip install scikit-build-core cmake ninja` then `pip install -e
  ".[full-dev]" --no-build-isolation -C editable.rebuild=true`.  The
  import-time `cmake --build` runs after pip exits, so the toolchain must
  persist in the venv; `--no-build-isolation` also skips
  `build-system.requires`, so the backend must be installed by hand first.
  An editable install that opted into `rebuild` with a pip-deleted toolchain
  raises on *every* `import qarp`.
- Cold builds on a < 8 GB box need `export CMAKE_BUILD_PARALLEL_LEVEL=3`, or
  `cc1plus` is OOM-killed compiling SymEngine.
- Python tests: `pytest` (defaults exclude `slow`/`bench` markers; full run
  ≈ 8 min).  C++ tests: `python scripts/run_cpp_tests.py` (CI: the `ctest` job).
- Notebooks: `pytest --nbmake examples/` needs `pip install -e ".[notebooks]"`
  (CI: the `notebooks` job, nightly).  Under a `-C editable.rebuild=true` install,
  export `SKBUILD_EDITABLE_VERBOSE=0` first: the import-time rebuild streams to
  a Jupyter kernel's stdout, which has no `fileno()`, so **every** notebook
  dies at `import qarp` with `UnsupportedOperation`.
- Lint + format, blocking in CI and pre-commit (config in `[tool.ruff]`):
  `ruff check --fix . && ruff format .`
- Types: `mypy qarp/ tests/ --disable-error-code=import-untyped
  --disable-error-code=method-assign`
- Git worktrees: a shared venv resolving `qarpx` to another checkout's build
  now fails fast at `import qarp` (ABI + source-dir stamps, `qarp/_abi.py`).
  The fix is a venv + editable install inside the worktree;
  `QARP_SKIP_ABI_CHECK=1` bypasses for deliberate cross-checkout runs.

## Landmines (each has caused a real bug)

- **Endianness**: everything in qarp is LSB (qubit 0 = least significant
  bit) — statevectors, sampler keys, ONVs *and* operator matrices
  (`op.sparse_matrix()`).  MSB exists only at external boundaries:
  openfermion / cirq / pennylane matrices (`qarp.operators.compat.
  get_sparse_operator` is openfermion's MSB layout, interop only) and
  quimb's kron-ordered `from_dense`.  Contracting across such a boundary
  needs a bit reversal (`qarp.endianness`); inside qarp it never does.
- **Angles are radians**, convention `exp(-iθP/2)` (§1–§12).  Legacy
  half-turn inputs are a recurring source of silent factor-π bugs.
- **Modulo-global-phase is not exact**: a block's global phase becomes a
  *relative* phase under `ControlledBlock` (QPE read shifted eigenphases from
  an uncalibrated synthesis).  Controllable blocks must be phase-exact and
  their unitary oracles compare exact equality (§13, §18).
- **`.symbols` is a canonically sorted tuple** (sorted by `str`) on every
  built block.  Never hand-zip parameter vectors against any other list —
  use `block.parameter_map(values)` / `optimal_parameters` (§17).
- **Block inheritance**: new internal blocks extend `SimpleBlock` (leaf) or
  `CompositeBlockBase` (tree).  The `Block` alias is gone — annotate "any
  block" with `AnyBlock` (= `qx.Block`); it is a type, not a base class.
- **SDK imports are integration-only** — qiskit/pytket/pennylane/qulacs never

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenQARP/openqarp](https://github.com/OpenQARP/openqarp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-14 -->
