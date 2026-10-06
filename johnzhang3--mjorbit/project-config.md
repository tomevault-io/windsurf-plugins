---
trigger: always_on
description: `src/mjorbit/` is the stable CPU reference backend. `src/mjorbit_warp/` is the
---

# Repository Guidelines

## Project Structure & Module Organization
`src/mjorbit/` is the stable CPU reference backend. `src/mjorbit_warp/` is the
GPU-targeting MJWarp backend. Both expose MuJoCo-style model/data APIs, and users choose the
backend by import path.

- `src/mjorbit/config.py` defines public `OrbitInit` and `*Spec` dataclasses.
- `src/mjorbit/model.py` defines the Python `MjoModel` wrapper.
- `src/mjorbit/data.py` defines the Python `MjoData` wrapper.
- `src/mjorbit/step.py` defines `mjo_forward` and `mjo_step`.
- `src/mjorbit/rollout.py` defines state and rollout helpers.
- `src/mjorbit/planning/` defines the spline-knot MPPI planner on the batched rollout.
- `src/cpp/` contains the language-neutral C++ core and MuJoCo plugin.
- `src/cpp/src/runtime_model.cc` compiles XML and resolves model metadata.
- `src/cpp/src/runtime.cc` owns per-`MjoData` allocation, reset, and frame conversions.
- `src/cpp/src/runtime_sensors.cc` owns C++ sensor bias/noise measurement helpers.
- `src/cpp/src/runtime_state.cc` owns stepping, state packing, and rollout calls.
- `src/cpp/bindings/` contains the nanobind Python module.
- `tests/mjorbit/reference/orbit/` contains Python reference/analysis orbit helpers.
- `tests/mjorbit/reference/coupling/` contains Python reference/analysis coupling helpers.
- `tests/mjorbit/reference/sensors.py` contains Python reference sensor helpers.
- `src/mjorbit/testdata/` contains XML fixtures used by tests and examples.
- `src/mjorbit_warp/` contains the optional MJWarp runtime, sync wrappers, and step API.
- `src/viewer/` contains the browser viewer integration and the `mjo-viewer` task app
  (`pixi run viewer`); built-in tasks live in `src/viewer/tasks/`.
- `examples/` contains paper example scenarios and small runnable demos of the public API.
- `experiments/` contains the paper figure generators.
- `scripts/record/` contains the headless viewer renderer for paper clips/stills.
- `tests/mjorbit/` mirrors the production package; shared setup lives in `_helpers.py`.
- `tests/mjorbit_warp/` holds MJWarp tests; guard them with `pytest.importorskip`.

## Build, Test, and Development Commands
Use `pixi` for environment management and command execution.

- `pixi install`: install the default Python 3.12 dev environment and editable package.
- `pixi install -e py311`: install the Python 3.11 dev environment.
- `pixi install -e report`: add report/analysis plotting dependencies.
- `pixi install -e warp`: add MJWarp and Warp for the GPU backend.
- `pixi run test`: run the full test suite.
- `pixi run test-api`: run the public API tests.
- `pixi run test-warp`: run the MJWarp tests when the optional extra is installed.
- `pixi run lint`: run linting and import-order checks.
- `pixi run typecheck`: run static type checks.
- `pixi run cpp-test`: configure, build, and test the native C++ plugin.
- `pixi run example-free-drift`: run a minimal API example.

## Coding Style & Naming Conventions
Follow existing Python style: 4-space indentation, explicit type hints, and small focused
modules. Ruff enforces a 100-character line length and import sorting.

Use `snake_case` for functions and modules, `PascalCase` for classes/dataclasses, `*Spec`
for static compile-time metadata, and `OrbitInit` for orbital initial conditions.

Do not reintroduce the removed compatibility layer. New public code should target
`MjoModel`, `MjoData`, `mjo_forward`, and `mjo_step` directly. Prefer
`model.make_data(...)` as the runtime construction path across both backends. Keep
`MjoData(model, orbit=...)` only as CPU compatibility support, not as the preferred pattern
for new code. Avoid adding new `Scenario`, `compile`, `step`, or `*Cfg` surfaces unless the
user explicitly asks for a new compatibility wrapper.

For warp work, preserve exact MuJoCo frame conventions and keep warp-specific code under
`src/mjorbit_warp/`. Backend choice should stay explicit by import path rather than by
auto-detection. Warp users request batched parallel simulations with `model.make_data(..., nworld=N)`.

## Testing Guidelines
Add tests in `tests/mjorbit/test_*.py` or `tests/mjorbit_warp/test_*.py` beside the
subsystem you change. Prefer deterministic numeric assertions with `numpy.testing` or
`pytest.approx`, and cover both nominal behavior and validation errors.

If you touch the runtime API, stepping, sensors, or coupling code, update
`tests/mjorbit/test_api_model_data.py` and the relevant subsystem tests. Run
`pixi run test` before submitting. If you touch warp code, add or update guarded tests in
`tests/mjorbit_warp/` as well. If you touch `experiments/` analysis scripts, run the
affected script directly and keep generated plots or figures intentional.

## Commit & Pull Request Guidelines
Use short imperative commit subjects such as `Add reaction wheel saturation test`. Keep
commits scoped to one change. PRs should include a concise summary, the commands you ran,
and any remaining issues.

Link the related issue when available. Include screenshots or short recordings for viewer
changes, and include plot diffs or artifact notes when the change intentionally updates
paper figure material under `experiments/`.

## Environment Notes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [johnzhang3/mjorbit](https://github.com/johnzhang3/mjorbit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
