---
trigger: always_on
description: SOLWEIG (SOlar and LongWave Environmental Irradiance Geometry-model) is a high-performance urban microclimate model. It computes mean radiant temperature (Tmrt) and thermal comfort indices (UTCI, PET) in complex urban environments.
---

# CLAUDE.md — SOLWEIG Project Guide

## What Is This Project?

SOLWEIG (SOlar and LongWave Environmental Irradiance Geometry-model) is a high-performance urban microclimate model. It computes mean radiant temperature (Tmrt) and thermal comfort indices (UTCI, PET) in complex urban environments.

- **Hybrid Rust/Python** — heavy compute in Rust (PyO3), orchestration and I/O in Python
- **Optional GPU acceleration** — wgpu compute shaders for shadow casting, SVF, GVF, and anisotropic sky
- **Dual geospatial backend** — rasterio (standalone) or GDAL (QGIS plugin), detected lazily at runtime
- **License**: GPL-3.0
- **Version**: 0.1.0 beta series (single source of truth: `pyproject.toml`)

## Read first — architectural anchors

Before making any non-trivial change, skim:

- **[PRINCIPLES.md](PRINCIPLES.md)** — what this
  library is for, the four identities it serves, the architectural rules that
  follow. When facing a "should this go here or there?" question, work from
  this page first.
- **[INVARIANTS.md](INVARIANTS.md)** — the
  load-bearing assumptions the code makes but does not always enforce
  (array layout, immutability, GIL ownership, etc). Violating these
  produces silently wrong results, not crashes.
- **[ARCHITECTURE.md](ARCHITECTURE.md)** — the layered overview (Python API
  → orchestration → fused Rust pipeline → Rust algorithms) and how data
  moves between them.
- **[ARCHITECTURE_REVIEW.md](ARCHITECTURE_REVIEW.md)** — the deeper review the
  principles and invariants pages derive from. Some numerical claims
  (e.g. line counts) are pre-b85 and now out of date; treat it as a
  snapshot of why the current architecture is shaped this way, not as
  live structural reference.

---

## Quick Reference: Common Commands

```bash
# Environment setup
uv sync --group test --group dev      # Install all dev + test deps
maturin develop --release             # Build Rust extension (MUST be --release)

# Linting & formatting
poe lint                              # ruff format + ruff check --fix
poe typecheck                         # ty check (NOT mypy)

# Testing
poe test_quick                        # pytest -m 'not slow' -x -q  (fast gate)
poe test_full                         # Full suite including slow tests
poe test_benchmarks                   # Performance regression tests
poe test_gpu_gates                    # GPU vs CPU parity tests
pytest tests/spec/ -x -q             # Scientific property tests only
pytest tests/golden/ -x -q           # Golden regression tests only
pytest tests/validation/ -x -q       # Real-world site validation

# Full verification (lint + typecheck + all tests)
poe verify_project

# Documentation
poe docs                              # mkdocs serve (local preview)
poe docs_build                        # mkdocs build --strict
```

---

## Repository Layout

```
pysrc/solweig/          Python package
  api.py                Main entry point: calculate()
  _compat.py            Backend detection (rasterio vs GDAL) — SINGLE SOURCE OF TRUTH
  io.py                 Raster I/O (branches on GDAL_ENV)
  computation.py        Core computation orchestration (calls into Rust pipeline)
  timeseries.py         Multi-timestep computation
  tiling.py             Large-raster tiling
  summary.py            Output summary generation
  loaders.py            Config/EPW file loaders (RENAMED from config.py)
  models/               Dataclasses: SurfaceData, Weather, Location, ModelConfig, etc.
  physics/              Pure-Python algorithms (RENAMED from algorithms/)
  components/           Ground, GVF, shadows, SVF resolution, 2026a ground-scheme init
  errors.py             Custom exception hierarchy (all inherit SolweigError)
  constants.py          Physical constants (Stefan-Boltzmann, view factors, etc.)
  solweig_logging.py    Logging (NOT logging.py — watch for stale refs)
  _orchestration.py     Internal orchestration helpers
  bundles.py            Data bundle types
  output_async.py       Async output writing
  buffers.py, cache.py, progress.py, walls.py, utils.py, metadata.py, postprocess.py
  data/                 Default JSON configs (params, physics, materials)

rust/src/               Rust extension (PyO3, compiled as solweig.rustalgos)
  lib.rs                PyModule root — 12 submodules, 60+ functions
  pipeline.rs           Fused per-timestep pipeline (single FFI call)
  shadowing.rs          Shadow casting (CPU + GPU)
  skyview.rs            Sky View Factor
  gvf.rs, gvf_geometry.rs  Ground View Factor + geometry caching
  sky.rs                Anisotropic sky radiation (Perez model)
  vegetation.rs         Tree effects (longwave + shortwave)
  ground.rs             Ground temperature + thermal delay
  ground_surface.rs     UMEP 2026a ground scheme (force-restore/OHM + outgoing-LW march, opt-in)
  utci.rs, pet.rs       Thermal comfort indices
  tmrt.rs               Mean Radiant Temperature
  perez.rs              Perez diffuse sky model math
  sun.rs                Sun-on-surface calculations
  wall_aspect.rs        Wall orientation detection (Goodwin filter)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [UMEP-dev/solweig](https://github.com/UMEP-dev/solweig) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
