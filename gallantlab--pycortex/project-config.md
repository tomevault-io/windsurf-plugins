---
trigger: always_on
description: Guidance for AI coding agents working in this repository. Claude Code reads this file through [CLAUDE.md](CLAUDE.md).
---

# AGENTS.md

Guidance for AI coding agents working in this repository. Claude Code reads this file through [CLAUDE.md](CLAUDE.md).

Pycortex visualizes fMRI and other volumetric neuroimaging data on cortical surfaces. It has two rendering paths (static matplotlib flatmaps and an interactive WebGL browser viewer) built on a shared subject database, data-container, and volume-to-surface mapping core.

## Commands

### Build / install

The package has two Cython extensions (`cortex/formats.pyx` for surface mesh I/O, `cortex/openctm.pyx` wrapping the vendored `OpenCTM-1.0.3/` C library), so `import cortex` fails from a source checkout until they are built:

```bash
uv sync --all-extras                              # headless extra (playwright) + dev group (test/type-checking deps) by default; builds the Cython extensions
uv run playwright install --only-shell chromium   # browser for headless WebGL tests
```

Versioning is by `setuptools-scm` from git tags (written to `cortex/_version.py`); never edit version files by hand. A new git worktree gets its own `.venv` — rerun `uv sync` there to (re)build the compiled extensions.

### Tests

```bash
uv run pytest                                              # full suite (pytest.ini: testpaths=cortex, coverage on)
uv run pytest cortex/tests/test_quickflat.py               # one file
uv run pytest cortex/tests/test_quickflat.py::test_make_figure  # one test
uv run pytest -p no:cacheprovider --no-cov -x cortex/tests/test_formats.py  # quick iteration without coverage
```

- `pytest.ini` sets a 240 s per-test timeout (via `pytest-timeout`) because headless browser sessions can hang; override per-test with `@pytest.mark.timeout(N)`.
- Tests self-skip based on available tools: Inkscape (quickflat/dataset overlay tests), Playwright Chromium (`cortex/tests/testing_utils.py:has_playwright` — WebGL headless tests), and FreeSurfer's `mri_surf2surf`.
- Nearly all tests use the stub subject `S1` bundled in `filestore/db/S1` (transform `fullhead`, 304380 vertices, volume shape `(31, 100, 100)`).
- CI (`.github/workflows/run_tests.yml`) runs `pytest --cov=./` on a matrix of Python versions, with Inkscape and Playwright Chromium installed. The only enforced lint is codespell (config in `pyproject.toml`). mypy is configured in `pyproject.toml` and installed with the dev group but not run in CI — new Python code should still carry type annotations.

### Docs

```bash
uv pip install sphinx_gallery numpydoc   # not (yet) a dependency group
cd docs && uv run make html              # gallery examples need Inkscape and Playwright too
```

## Architecture

### Subject database ("filestore")

`cortex/database.py` defines `Database`; the module-level singleton `cortex.db` is the entry point everything else uses (`db.get_surf`, `db.get_xfm`, `db.get_overlay`, `db.get_mask`, attribute access like `db.S1.surfaces.fiducial`). The on-disk layout is defined entirely by `Database.get_paths`: each subject dir holds `surfaces/` (GIFTI, `{type}_{lh|rh}.gii`), `transforms/{xfmname}/` (affine + reference nifti + masks), `anatomicals/`, `surface-info/` (cached npz), `overlays.svg`, `views/`, and `cache/`. The filestore location comes from the user config (see below); the repo bundles a minimal filestore with the S1 stub subject and the colormap PNGs.

### Data containers — `cortex/dataset/`

Two orthogonal mixin axes: **braindata** (the array plus subject/transform identity: `VolumeData`, `VertexData` in `braindata.py`) and **Dataview** (display state — cmap, vmin/vmax — plus JSON/HDF5 serialization, in `views.py`). Public classes combine them: `Volume`, `Vertex`, `Volume2D`/`Vertex2D` (`view2D.py`), `VolumeRGB`/`VertexRGB` (`viewRGB.py`). `Dataset` (`dataset/dataset.py`) is an HDF5-backed dict of dataviews (`cortex.load` = `Dataset.from_file`). `dataset.normalize()` coerces loose inputs — a `(data, subject, xfmname)` tuple or dict of arrays — into dataviews, which is why every plotting function accepts them.

### Volume ↔ surface mapping — `cortex/mapper/`

`get_mapper(subject, xfmname, type=...)` builds per-hemisphere sparse matrices (point/line/patch sampling families × nearest/trilinear/gaussian/lanczos kernels) and caches them in the subject's `cache/` dir. `Mapper.__call__` maps volume→vertex; `.backwards` goes the other way. Cache invalidation is mtime-based against the `.xfm` file only — editing surfaces does not invalidate it; `db.clear_cache(subject)` is the escape hatch.

`cortex/xfm.py` (`Transform`) handles the affine conventions, including `from_fsl`/`to_fsl` and `from_freesurfer`/`to_freesurfer` conversions.

### Static flatmaps — `cortex/quickflat/`

`quickflat.make_figure` (= `cortex.quickshow`) orchestrates the layer functions in `composite.py` (`add_curvature`, `add_data`, `add_rois`, `add_sulci`, `add_colorbar`, …). `utils.make_flatmap_image` rasterizes the flat surface, with pixel/vertex caches in the subject cache dir. Rendering ROI/sulci layers shells out to Inkscape.

### WebGL viewer — `cortex/webgl/`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gallantlab/pycortex](https://github.com/gallantlab/pycortex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
