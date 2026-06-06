---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

OpenTrustRegion is a Fortran library implementing second-order trust region orbital optimization for quantum chemistry, exposed via Fortran, C, and Python (ctypes) interfaces. The same compiled shared/static library is consumed by all three.

## Project priorities

**Clarity over performance.** The library is not on the performance hot path of a quantum-chemistry calculation — the host program's integral transforms and Hessian-vector products dominate. Prefer code that is short and obviously correct over code that is fast. Don't propose performance refactors (amortized buffer growth, pooling, micro-optimizations) unless there is evidence the affected code is hot for a real workload.

**Fortran/C/Python interfaces must always be consistent.** The same callback signatures, settings fields, and default values are described in six places that must agree:

1. Fortran abstract interfaces and `solver_settings_type` / `stability_settings_type` in `src/opentrustregion.f90`
2. C abstract interfaces and `bind(C)` `solver_settings_type_c` / `stability_settings_type_c` in `src/c_interface.f90`
3. C struct layouts and typedefs in `include/opentrustregion.h`
4. `SolverSettingsC` / `StabilitySettingsC` ctypes `_fields_` and `CFUNCTYPE` declarations in `pyopentrustregion/python_interface.py`
5. `default_solver_settings` / `default_stability_settings` (Fortran) ↔ `solver_settings_init` / `stability_settings_init` (C) ↔ the Python `Settings` wrapper defaults
6. The argument lists and snippets in `README.md`

Any change to a callback signature, a settings field (add / remove / reorder), or a default value must land in all six locations in the same PR. Additionally, the error-origin codes (`error_obj_func` etc. in `opentrustregion.f90`) must be synchronized with the README error-code table.

## Build & test

```sh
# Fortran/C build
mkdir build && cd build
cmake ..              # add -DBUILD_SHARED_LIBS=ON for shared
cmake --build .

# Python install (invokes CMake under the hood via setup.py)
pip install .
pip install -e .       # editable
```

Run the full test suite (Python ctypes driver that calls into `libotrtestsuite`, which contains the Fortran unit + system tests):

```sh
python3 -m pyopentrustregion.testsuite                  # from an installed/editable build
python3 pyopentrustregion/testsuite.py                  # from the source tree against ./build
```

Run a single test class or method using stdlib `unittest`:

```sh
python3 -m unittest pyopentrustregion.testsuite.SystemTests
python3 -m unittest pyopentrustregion.testsuite.OpenTrustRegionUnitTests.test_solver
```

The Python suite runs Fortran- and C-side tests (via symbols dynamically loaded from `libotrtestsuite`) and pure-Python wrapper tests. System tests need `pyopentrustregion/test_data/*.bin`.

### CMake options that matter

- `INTEGER_SIZE` (`4` or `8`): library integer width. If unset, CMake autodetects by trying 32-bit BLAS/LAPACK first, falling back to 64-bit. Output library is named `libopentrustregion_32.*` or `libopentrustregion_64.*`. Defining `USE_ILP64` (set automatically when `INTEGER_SIZE=8`) switches both Fortran `ip` and the C `c_ip` typedefs to 64-bit and remaps BLAS/LAPACK symbol names to their `_64` variants when `check_fortran_function_exists` finds them.
- `BLAS_LIBRARIES` / `LAPACK_LIBRARIES`: required to be set together with `INTEGER_SIZE` if overriding autodetection — passing one without the other is a fatal error.
- `OpenTrustRegion_BUILD_TESTING` (default ON when top-level): builds `libotrtestsuite`, the shared library Python loads to drive the Fortran tests.
- `CONDA_BUILD=1` env var: the Python `setup.py` skips the embedded CMake invocation (the conda recipe builds the C library separately).

## Architecture

### Source layout

- `src/opentrustregion.f90` — the entire numerical core: `solver`, `stability_check`, the Davidson / Jacobi-Davidson / truncated-CG subsystem solvers, settings derived types, callback abstract interfaces, error-code constants. ~2.6k lines, single module.
- `src/c_interface.f90` — `bind(C)` wrapper module. Stores Fortran procedure pointers to C callbacks at module scope (`update_orbs_before_wrapping`, etc.) and adapts C-style `(*)` arrays + return-code functions into the Fortran-style `(:)` arrays + `intent(out) :: error` subroutines that the core expects.
- `include/opentrustregion.h` — C header mirroring the Fortran settings types (`solver_settings_type`, `stability_settings_type`) as C structs plus `*_init()` helpers and `solver` / `stability_check` prototypes.
- `pyopentrustregion/python_interface.py` — ctypes wrapper. Defines `SolverSettings` / `StabilitySettings` as `ctypes.Structure` mirrors of the C structs, wraps Python callbacks with `CFUNCTYPE`, and converts the integer error codes returned by the C entry points into `RuntimeError`.
- `tests/` — three layers:
  - `opentrustregion_unit_tests.f90` / `c_interface_unit_tests.f90` — unit tests against the Fortran and C interfaces respectively, both built into `libotrtestsuite`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eriksen-lab/opentrustregion](https://github.com/eriksen-lab/opentrustregion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
