---
trigger: always_on
description: Canonical navigation and conventions guide for the pylinac repository, used by Codex and Cursor agents.
---

# AGENTS.md: Pylinac Codebase Guide

## Purpose & Scope

Canonical navigation and conventions guide for the pylinac repository, used by Codex and Cursor agents.

Pylinac is an open-source Python library for performing TG-142 QA-related image analysis on linear accelerators. The typical user is a medical physicist without computer science training, so APIs must remain simple, well-documented, and backwards-compatible.

## Directory Intent Map

| Path | Intent | Primary Entry Points | Tests / Validation |
| --- | --- | --- | --- |
| `pylinac/` | Source package root | `pylinac/__init__.py` | `tests_basic/` |
| `pylinac/core/` | Shared infrastructure: image loading, geometry, profiles, ROI, PDF, MTF, gamma, NPS, utilities | `pylinac/core/image.py`, `pylinac/core/profile.py`, `pylinac/core/utilities.py` | `tests_basic/core/` |
| `pylinac/core/image_generator/` | Synthetic image generation for testing and demos | `simulators.py`, `layers.py`, `utils.py` | `tests_basic/core/test_image_generator.py` |
| `pylinac/metrics/` | Image and profile metric abstractions | `image.py`, `profile.py`, `features.py` | `tests_basic/core/test_image_metrics.py` |
| `pylinac/calibration/` | TG-51 and TRS-398 reference dosimetry | `tg51.py`, `trs398.py` | `tests_basic/test_tg51.py`, `tests_basic/test_trs398.py` |
| `pylinac/contrib/` | Community-contributed analysis modules | `quasar.py`, `orthogonality.py` | `tests_basic/contrib/` |
| `pylinac/plan_generator/` | DICOM RT plan generation (MLC, fluence) | `dicom.py`, `mlc.py`, `fluence.py` | `tests_basic/test_plan_generator.py` |
| `tests_basic/` | Primary test suite (pytest + unittest) | `tests_basic/test_*.py` | See `tests_basic/AGENTS.md` |
| `tests_bank/` | Bulk data-bank tests over large image sets | `_test_bank_*.py` | Requires local `pylinac test files` directory |
| `docs/` | Sphinx documentation tooling and source tree | `docs/source/conf.py` | `nox -s build_docs`; see `docs/source/AGENTS.md` |
| `scripts/` | Helper and maintenance scripts | scripts in directory | manual execution |
| `.github/workflows/` | CI/CD via GitHub Actions | `ci.yml`, `python-versions.yml` | automated |

## Analysis Module Pattern

All analysis classes follow a consistent workflow:

```
Load -> Analyze -> Results / Visualize
```

**Loading** (one of):
- `__init__(filepath)` -- from file path or directory
- `from_url(url)` -- from a URL
- `from_demo_image()` / `from_demo_images()` -- built-in demo data
- `from_zip(path)` -- from a zip archive

**Analysis**:
- `analyze(**kwargs)` -- module-specific parameters

**Results** (all of):
- `results()` -- human-readable string summary
- `results_data()` -- structured Pydantic model (`ResultBase` subclass)
- `plot_analyzed_image()` -- matplotlib visualization
- `plotly_analyzed_images()` -- Plotly interactive visualization
- `save_pdf(filename)` -- PDF report via ReportLab

## Key Base Classes and Mixins

| Class | Location | Purpose |
| --- | --- | --- |
| `ResultBase` | `pylinac/core/utilities.py` | Pydantic base model for all analysis results (version, date, warnings) |
| `ResultsDataMixin[T]` | `pylinac/core/utilities.py` | Generic mixin providing `results_data()`, `_generate_results_data()`, `as_dict`, `as_json` |
| `QuaacMixin` | `pylinac/core/utilities.py` | Integration with quaac QA metadata |
| `WarningCollectorMixin` | `pylinac/core/warnings.py` | Collects warnings during analysis |
| `ImagePhantomBase` | `pylinac/planar_imaging.py` | Base for all planar imaging phantom classes |
| `CatPhanBase` | `pylinac/ct.py` | Base for all CatPhan CT/CBCT phantom classes |
| `MetricBase` | `pylinac/metrics/image.py` | ABC for image-level metrics |
| `ProfileMetric` | `pylinac/metrics/profile.py` | ABC for profile-level metrics |

## Analysis Modules

| Module | File | Key Classes |
| --- | --- | --- |
| ACR Phantoms | `acr.py` | `ACRCT`, `ACRMRILarge` |
| CBCT / CatPhan | `ct.py` | `CatPhan503`, `CatPhan504`, `CatPhan600`, `CatPhan604`, `CatPhan700` |
| Cheese Phantoms | `cheese.py` | `CIRS062M`, `TomoCheese` |
| Quart DVT | `quart.py` | `HypersightQuartDVT`, `QuartDVT` |
| Planar Imaging | `planar_imaging.py` | `LasVegas`, `LeedsTOR`, `StandardImagingFC2`, `StandardImagingQC3`, `PTWEPIDQC`, and others |
| Picket Fence | `picketfence.py` | `PicketFence` |
| Starshot | `starshot.py` | `Starshot` |
| VMAT | `vmat.py` | `DRCS`, `DRGS`, `DRMLC` |
| Winston-Lutz | `winston_lutz.py` | `WinstonLutz`, `WinstonLutz2D`, `WinstonLutzMultiTargetMultiField` |
| Field Analysis | `field_analysis.py` | `FieldAnalysis`, `DeviceFieldAnalysis` |
| Field Profiles | `field_profile_analysis.py` | `FieldProfileAnalysis` |
| Machine Logs | `log_analyzer.py` | `Dynalog`, `TrajectoryLog`, `MachineLogs`, `load_log` |
| Nuclear | `nuclear.py` | Nuclear medicine QA |
| TG-51 | `calibration/tg51.py` | TG-51 reference dosimetry |
| TRS-398 | `calibration/trs398.py` | TRS-398 reference dosimetry |

---

## Development

### Package Management

- Use `uv` as the Python package manager instead of `pip`.
  - Do not install packages without prompting the user.
  - The Python virtual environment is in `.venv/`.
- Install with dev dependencies: `uv pip install -e ".[developer]"`

### Running Tests

- All tests: `uv run pytest`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jrkerns/pylinac](https://github.com/jrkerns/pylinac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
