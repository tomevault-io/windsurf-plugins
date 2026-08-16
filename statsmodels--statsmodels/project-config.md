---
trigger: always_on
description: Guidance for coding agents (and humans) contributing to `statsmodels`. This
---

# AGENTS.md

Guidance for coding agents (and humans) contributing to `statsmodels`. This
file is a practical supplement to [CONTRIBUTING.md](CONTRIBUTING.md) and
the [developer docs](https://www.statsmodels.org/devel/dev/index.html) —
read those too, but treat this as the checklist for "will this actually get
merged."

## Project snapshot

- Pure-Python core with a few Cython extensions (Kalman filter code in
  `statsmodels/tsa/statespace/_*.pyx`, etc.). Most new statistical code is
  pure Python.
- Cython extensions are built via `meson`/`ninja` (see
  `pyproject.toml` `[tool.meson-python]` section). Cython extensions
  are accepted only when the performance gain over pure Python is significant.
- Build backend is `meson-python`; the package is normally developed via an
  **editable install** so pure-Python edits are picked up immediately.
- Target Python: `py310`+ (see `[tool.ruff] target-version` /
  `[tool.black] target-version` in `pyproject.toml`).
- Upstream repo is `statsmodels/statsmodels`; when linking issues/PRs in
  commit messages or docs, link that repo, not a personal fork.
- Ensure features taken from upstream packages, `numpy`, `scipy`, `pandas`, etc.,
  are compatible with the minimum versions in `pyproject.toml` and `requirements.txt`.

## Environment setup

```bash
python -m pip install -r requirements.txt
python -m pip install -r requirements-dev.txt
python -m pip install -e . --no-build-isolation
```

Using an editable install mean changes in both Python and Cython are live
immediately; Cython extensions are rebuilt automatically on import
if the source `.pyx` file is newer than the compiled `.so` or `.dll` file.

## Code style & linting

- Formatting: `ruff` (line-length 88, `py310` target, isort-equivalent import sorting via
  `[tool.ruff.lint.isort]`). Import sections: `future`, `compat`,
  `standard-library`, `third-party`, `first-party`, `local-folder` — with
  `statsmodels.compat` treated as its own `compat` section.
- `ruff check .` and `flake8 .` must pass when run from the repo root.
- `black` (line-length 88) is acceptable for auto-formatting, but not required.
  Only use `black` to format changes to files, or in new files. Do not use `black` to
  wholesale reformat existing files.

## Docstrings & documentation

- Every public function, class, method, and attribute needs a numpydoc
  docstring (numpy docstring standard, not Google/Sphinx style).
- New public functionality must be:
  1. Exported from the relevant `api.py` if the module has one (e.g. new
     `tsa` models go in `statsmodels/tsa/api.py`, matching how `VAR`,
     `SVAR`, `VARMAX` are exposed there) — burying a new class only inside
     an internal module path is a common reason reviewers bounce a PR.
  2. Added to the Sphinx docs — find the relevant `docs/source/*.rst` file
     (e.g. `docs/source/vector_ar.rst` for VAR-family models) and add an
     `autosummary` entry.
  3. Noted in the current release notes file under `docs/source/release/`
     (currently `version0.15.0.rst` — check for the highest-numbered
     unreleased `versionX.Y.Z.rst`).
- If it's a substantial new feature, an example notebook is strongly encouraged.
  (see the "Add an example" step in `CONTRIBUTING.md`).

## Testing conventions

- Tests live in a `tests/` subpackage next to the code
  (`statsmodels/<area>/tests/test_*.py`). Reference values from other
  packages (R, Stata, SAS, a prior statsmodels version) go in a sibling
  `tests/results/` directory — document *where* each reference value came
  from and why it might not match exactly.
- The preferred style is pure modern `pytest` with `pytest.mark.parametrize`
  for multiple cases, and `pytest` fixtures for shared setup.
- Avoid `unittest.TestCase` style for new tests.
- Statistical/numerical claims should be validated against an existing
  package or known closed-form result wherever possible (this project
  follows a test-driven-development norm for new models/statistical
  functions) — an internal-consistency-only test suite (e.g. "shape is
  right", "output is finite") is necessary but not sufficient.
- Two sharp edges that will make a locally-green test suite fail once
  merged — **verify against the actual `pytest` run in this repo, not just
  by reading the code**:
  - **Global RNG state is protected.** `statsmodels/conftest.py` has an
    autouse fixture (`check_global_randomstate_usage`) that asserts the
    legacy global `np.random` singleton state is unchanged after every
    test. Use a seeded `np.random.default_rng(seed)` `Generator` in new
    tests, not bare `np.random.randn(...)`/`np.random.seed(...)`. If a test
    genuinely must touch the global singleton, mark it
    `@pytest.mark.singleton_randomstate`.
  - **Many warnings are promoted to errors.** `pyproject.toml`'s
    `[tool.pytest.ini_options] filterwarnings` turns a long list of
    `DeprecationWarning`/`FutureWarning` patterns into hard test failures.
    New code must not exercise deprecated pandas/numpy/scipy code paths,
    and should assume that list grows over time — a warning that's merely
    "ignored" today may be promoted to `error` later.
  - Other markers worth knowing: `slow`, `example`, `matplotlib`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [statsmodels/statsmodels](https://github.com/statsmodels/statsmodels) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
