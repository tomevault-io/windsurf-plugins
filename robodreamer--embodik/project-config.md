---
trigger: always_on
description: Context for AI agents and contributors to pick up work efficiently. See `README.md` for user-facing docs and `docs/` for detailed guides.
---

# EmbodiK — Agent Context

Context for AI agents and contributors to pick up work efficiently. See `README.md` for user-facing docs and `docs/` for detailed guides.

## TL;DR

- **Build/test/release/gotchas:** See Quick Commands and Common Gotchas below.
- **Core layout:** C++ in `cpp_core/`, Python bindings in `python_bindings/`, examples in `examples/`
- **CoM constraints:** `configure_com_constraint()` with polygon vertices; margin uses `char_size` (mean centroid→vertex distance)
- **ECTS dual-arm:** `add_absolute_frame_task()`, `add_relative_frame_task()`, `add_frame_task()`; `map_ects_mode()` / `map_ects_mode_blended()`; reference: H. A. Park, IROS 2016 (doi:10.1109/IROS.2016.7759161)
- **Private context:** maintainer-only notes, local paths, and private integration details live outside this public repository.

---

## Agent Workflow (Recommended)

1. **Scope first**: confirm task target (`bug`, `feature`, `docs`, `release`) and touched paths.
2. **Load context**: check `AGENTS.md`, public docs in `docs/`, and the relevant source/tests.
3. **Implement minimally**: prefer localized diffs over broad rewrites; preserve existing APIs unless requested.
4. **Validate**: run the smallest relevant checks first, then broader checks.
5. **Document**: update public docs for user-facing behavior; keep maintainer-only reasoning outside this repository.
6. **Keep private context private**: do not commit local paths, private integration details, branch-cleanup logs, or customer/company-specific investigation notes.

Definition of done (default):

- Relevant tests pass (`pixi run test` or targeted subset)
- Build passes (`pixi run build`) for C++/bindings changes
- Docs build passes (`pixi run docs-build`) if docs changed
- Any new behavior is reflected in `docs/` and/or examples when user-facing

---

## Coding & Documentation Guardrails

Apply these by default unless the user asks otherwise:

- **Design**: prefer DRY/KISS/YAGNI; keep separation of concerns (solver core vs adapters/UI).
- **API safety**: preserve stable public behavior; use additive changes when possible.
- **Readability**: choose descriptive names; keep functions focused; refactor opportunistically when low risk.
- **Typing/logging**: add type hints in Python changes; prefer `logging` over ad-hoc `print` in library code.
- **Dependencies**: reuse existing project/tooling patterns before introducing new frameworks.
- **Performance**: profile/measure before optimizing; avoid speculative complexity.

Docstring and docs quality:

- Follow existing lint/style conventions in repo configs.
- Write concise, audience-aware docs (what/why/how) with runnable examples where relevant.
- Include troubleshooting notes for non-obvious failure modes.
- Keep terminology consistent across API docs, examples, and guides.

Naming/file organization:

- Follow existing naming patterns in the touched area first (do not mass-rename to a new scheme).
- For new module families, keep category/variant naming consistent within that directory.

Testing expectations:

- Prefer deterministic, targeted tests for changed behavior.
- For numerical checks, use tolerances and dtypes that match current test patterns.
- Add regression tests for any fixed bug that was user-visible or release-impacting.

---

## Repo Map

| Path | Purpose |
|------|---------|
| `cpp_core/include/embodik/` | C++ headers (KinematicsSolver, RobotModel, tasks, dual_arm_ects) |
| `cpp_core/src/` | C++ implementation (kinematics_solver.cpp, robot_model.cpp, dual_arm_ects.cpp) |
| `python_bindings/src/` | Nanobind bindings (kinematics_solver_bindings.cpp, tasks_bindings.cpp, etc.) |
| `python/embodik/` | Python package (gpu, examples, utils) |
| `examples/` | Standalone example scripts, including basic IK, collision-aware IK, CoM, dual-arm ECTS, whole-body bimanual, and G1 retargeting demos |
| `examples/utils/` | dual_iiwa_urdf.py, dual_panda_urdf.py, robot_models.py |
| `test/` | Pytest suite |
| `scripts/` | Build helpers (version.py, patch_qhull_cmake.py, upload_pypi.sh) |
| `docs/` | MkDocs documentation |

---

## Private Maintainer Notes

Maintainer-only notes are intentionally kept outside this public repository.
Do not add private investigation logs, local workspace paths, company/customer integration details,
branch-cleanup maps, or private benchmark artifacts to this repo.

If private context is needed to answer a public issue or implement a public change, ask the maintainer
for the minimal public-safe excerpt and document only the reusable behavior in `docs/`, examples, or
tests.

---

## External/Private Reference Policy

Some useful references may be private or not publicly crawlable (for example, internal
PDFs, private repos, or non-indexed docs). In those cases:

- Do not assume access to non-public repositories.
- Ask for concrete excerpts when precision matters (API signatures, constraints, workflows).
- Ask the maintainer for public-safe excerpts; do not copy private/internal material into this repository.
- Avoid copying private/internal language into public docs unless explicitly intended.

---

## Quick Commands & Release Workflow

Always run project commands through Pixi:

```bash
pixi run install

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robodreamer/embodik](https://github.com/robodreamer/embodik) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
