---
trigger: always_on
description: This is the PySCFAD repository.
---

# CLAUDE.md

## What This Is

This is the PySCFAD repository.
The repo contains three packages:
- **pyscfad** - the pure-Python differentiable (via JAX by default) quantum chemistry library (see `pyproject.toml`).
- **pyscfadlib** - a C/C++ support library with custom primitives and vjp rules (see `pyscfadlib/pyproject.toml`).
- **pyscfad-cuda12-plugin** / **pyscfad-cuda13-plugin** - C++ plugins (one per CUDA major) interfacing with third-party CUDA libraries, built with CMake (see `pyscfadlib/plugins/cuda/`).

## Building

### Build Commands

- **pyscfadlib**: `pip install pyscfadlib`
  - **Build from source**:
    ```bash
    cd pyscfadlib
    pip install .
    ```

- **pyscfad-cuda12-plugin** / **pyscfad-cuda13-plugin**: `pip install pyscfad-cuda12-plugin`
  (or `pyscfad-cuda13-plugin`)
  - **CRITICAL**: only needed when running on NVIDIA GPUs.
  - **Build from source** (CMake): needs a matching CUDA toolkit on `PATH`
    (CUDA 12.8+ for the cuda12 wheel, 13.x for cuda13) plus the `cmake`, `nanobind`,
    `jax`, and `build` Python packages. From `pyscfadlib/`:
    ```bash
    python plugins/cuda/build_plugin.py --cuda-major 13   # or --cuda-major 12
    pip install dist/pyscfad_cuda13_plugin*.whl
    ```
    `build_plugin.py` drives `plugins/cuda/CMakeLists.txt`, which builds the `_solver`
    (cuSOLVER) and `_cuint` nanobind modules and fetches the `cuint` kernels from GitHub.
    Device archs default to up to `sm_120` for the CUDA major; override with `--cuda-arch`.

- **pyscfad**: `pip install pyscfad`
  - **Build from source** (requires pyscfadlib build first; run from the repo root):
    ```bash
    pip install .
    ```

**CRITICAL**: do not set a timeout when building from source.

## Testing

### Running Tests

- **Single test file**: `pytest tests/test_scf.py`
- **Single module tests**: `pytest pyscfad/gto`
- **Full suite tests**: see `.github/workflows/run_test.sh`

Tests marked `_high_cost` or `_skip` are excluded by default (see `addopts` in
`pyproject.toml`). The `examples/` directory is ignored by pytest.

### Test Organization

**Default: add your test to existing test files unless new modules are added.**

When adding tests for a new module, place them under `tests/` by size:
- **Small** (a handful of tests): add a single file `tests/test_{module_name}.py`.
  - Example: tests for the `scf` module live in `tests/test_scf.py`; tests for `dft`
    live in `tests/test_dft.py`.
- **Larger** (multiple files / fixtures): create a subdirectory
  `tests/{module_name}/test_*.py`.
  - Example: the `xtb` module has `tests/xtb/` with `test_xtb.py`, `test_kxtb.py`,
    and `test_xtb_pad.py`.

Note the legacy layout: many modules also keep tests in-package at
`pyscfad/{module_name}/test/test_*.py` (e.g. `pyscfad/gto/test/`, `pyscfad/cc/test/`,
`pyscfad/df/test/`). These still run, but prefer the top-level `tests/` locations above
for new tests.

### Writing Tests

- **Reuse shared fixtures from `tests/conftest.py` first.** It already provides common
  molecule fixtures (e.g. `mol_H2`, `mol_H2O`, `mol_N2`) built via `tests/util.make_mol`.
  Prefer these over constructing molecules by hand so systems stay consistent and small.
- **Use `@pytest.fixture` whenever possible.** For anything not covered by
  `tests/conftest.py`, build shared objects (molecules, parameters, reference values) in
  fixtures rather than reconstructing them inside each test, and share them across a
  module's directory via a local `conftest.py` (see `tests/xtb/conftest.py` for the
  reference-value pattern, and the `setup` fixture in `tests/xtb/test_xtb.py`).
- **Keep tests small and fast.** A test should exercise one behavior with the smallest
  system that demonstrates it (e.g. a minimal basis, a couple of atoms). The default
  suite must stay quick — prefer cheap molecules and tight, deterministic assertions.
- **Name expensive tests with the `_high_cost` suffix.** Any test that is slow or
  resource-heavy (large systems, deep convergence, big derivatives) must end its name
  with `_high_cost`, e.g. `def test_ccsd_gradient_high_cost():`. These are excluded from
  the default run via `addopts` in `pyproject.toml` (`_skip` is similarly excluded). Keep
  the bulk of coverage in fast, default-run tests.

## Architecture

### Backend abstraction (`pyscfad/backend/`)

All array operations route through a swappable backend so the same chemistry code can
run under different AD/array engines. Selection is `jax` by default, configurable via
`PYSCFAD_BACKEND` env var or `~/.pyscfad/pyscfad.json` (allowed: `numpy`, `cupy`,
`jax`, `torch`; jax is the only fully supported one). Precision defaults to float64
(`PYSCFAD_FLOATX`).

Chemistry code should import arrays/ops through the backend, **not** directly from jax:
- `from pyscfad import numpy as np` → backend ndarray namespace (`pyscfad.backend.numpy`,
  resolved lazily via `__getattr__` to the active backend).
- `from pyscfad import ops` → AD/control-flow primitives: `stop_gradient`/`stop_grad`,
  `custom_jvp`, `jit`, `vmap`, `while_loop`, `index_update`/`index_add` (functional
  in-place updates), `to_numpy`, `is_array`.
- `pyscfad.backend.numpy.safe_sqrt` and similar guard against non-finite derivatives
  at singular points (e.g. `grad(sqrt)(0)`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fishjojo/pyscfad](https://github.com/fishjojo/pyscfad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
