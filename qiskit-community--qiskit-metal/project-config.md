---
trigger: always_on
description: > If you are a Claude (or other AI agent) opening this repo for the first
---

# CLAUDE.md — Quantum Metal repo guide for AI agents

> If you are a Claude (or other AI agent) opening this repo for the first
> time, **read this file end-to-end before touching anything**. It points
> at the rest of the context that will save you hours.

## What this repo is

**Quantum Metal** (formerly Qiskit Metal) is an open-source Python framework
for designing and analysing superconducting quantum chips. The PyPI package
is `quantum-metal`; the import path is still `qiskit_metal` for backward
compatibility. The community-maintained successor to IBM's original
Qiskit Metal — the rebrand is in progress through the v0.6.x line.

Stack: Python 3.10–3.12 · `shapely` for geometry · `geopandas` /
`pandas` for storage · `matplotlib` for headless viewing · `PySide6` for
the optional desktop GUI · `pyEPR-quantum` / `pyaedt` / `gmsh` /
`Elmer` for analysis backends.

## Architecture map (skim these first)

| Path | What lives there |
|------|------------------|
| `src/qiskit_metal/qlibrary/` | All `QComponent` subclasses (transmons, terminations, lumped, couplers, routes, sample shapes). The user-visible catalogue. |
| `src/qiskit_metal/qlibrary/core/base.py` | `QComponent` — the load-bearing base class. Read end-to-end before touching any component. |
| `src/qiskit_metal/designs/` | `QDesign` and subclasses (`DesignPlanar`, `DesignFlipChip`, ...). Components attach to a design. |
| `src/qiskit_metal/renderers/renderer_base/` | `QRenderer` and `QRendererAnalysis` — the two abstract bases. See `docs/architecture/renderer_protocol.md`. |
| `src/qiskit_metal/renderers/renderer_ansys/` | Legacy COM-based HFSS/Q3D renderer. Hard-touch zone. |
| `src/qiskit_metal/renderers/renderer_ansys_pyaedt/` | New pyaedt-based HFSS/Q3D renderer. Migration in progress. |
| `src/qiskit_metal/renderers/renderer_gds/` | `QGDSRenderer` — export to GDS. Pure-Python; safe to touch. |
| `src/qiskit_metal/renderers/renderer_mpl/` | The matplotlib renderer used by both the Qt GUI and `qm.view`. `QMplRenderer` no longer requires Qt as of v0.6.1. |
| `src/qiskit_metal/renderers/renderer_gmsh/`, `renderer_elmer/` | Open-source FEM path. Depends on `gmsh` (optional). |
| `src/qiskit_metal/viewer/` | New (v0.6.1) — `qm.view(design)` headless entry point. |
| `src/qiskit_metal/_gui/` | The Qt desktop GUI (`MetalGUI`). Hard-touch zone unless you have a Qt session to test in. |
| `src/qiskit_metal/analyses/` | Pure-Python analyses (Hamiltonian, capacitance, EPR). qutip 5+. |
| `tests/` | unittest-style suite. `pytest tests/` to run; gated in CI on every PR. |
| `tutorials/` | 40+ Jupyter notebooks. Numbered (1-Overview / 2-Components / 3-Renderers / 4-Analysis). **Mirrored 1:1 in `docs/tut/`** — see "Dual-folder tutorials" below. |
| `docs/tut/` | Sphinx-rendered copy of `tutorials/` with hyphenated filenames (so nbsphinx URLs work). Must stay content-identical to `tutorials/` — CI enforces. |
| `docs/` | Sphinx. `tox -e docs` to build. |
| `scripts/check_env_consistency.py` | CI gate that asserts `environment.yml` and `pyproject.toml` agree. |
| `scripts/check_tutorials_sync.py` | CI gate that asserts `tutorials/` and `docs/tut/` notebook cell content is byte-identical. |

## Dual-folder tutorials — read before editing notebooks

Every numbered notebook (1.x, 2.xx, 3.x, 4.xx) lives in **two places** that
must stay content-identical:

| Path                          | Why                                                         |
|-------------------------------|-------------------------------------------------------------|
| `tutorials/`                  | User-facing: GitHub browse, JupyterLab file tree open       |
| `docs/tut/` (hyphenated names)| Sphinx + nbsphinx source for the rendered docs site         |

**Editing one without the other silently breaks the docs site or the
notebook the user opens.** CI fails the PR if drift is detected
(`scripts/check_tutorials_sync.py` runs on every push/PR).

If you intentionally edit one folder, re-sync the other with:

```bash
python3 _dev/sync_two_folders.py --write
```

The sync script auto-detects which side has the uncommitted edit (vs
HEAD) and copies *that* side to the other. Editing `tutorials/` → propagates
to `docs/tut/`; editing `docs/tut/` → propagates to `tutorials/`. No
config required for the common case. The `CANONICAL` dict in that script
is **only** a tiebreaker for the rare case where both sides were edited
locally before sync — the default tiebreaker is `tut` (the user-facing
root). Re-run the CI gate to verify:

```bash
uv run scripts/check_tutorials_sync.py
```

**Why two folders, not one:** the naming constraints are mutually exclusive
— Sphinx + nbsphinx need hyphenated filenames for clean URLs and link
resolution, JupyterLab + GitHub-browse + external citations need the
human-readable space-separated form. There is no single naming scheme
that satisfies both. Drift is the failure mode; CI prevents it. Do not
"simplify" by deleting one folder.

## Hard constraints — do not touch without explicit human approval

1. **`renderers/renderer_ansys/`** — COM-based HFSS/Q3D. Requires Ansys
   AEDT on Windows to validate. Even type-comparison changes have
   shipped silent bugs.
2. **`renderers/renderer_ansys_pyaedt/`** — same constraint for the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qiskit-community/qiskit-metal](https://github.com/qiskit-community/qiskit-metal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
