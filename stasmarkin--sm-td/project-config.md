---
trigger: always_on
description: - `sm_td/`: Core C module for QMK (`sm_td.c`, `sm_td.h`, `qmk_module.json`).
---

# Repository Guidelines

## Project Structure & Module Organization
- `sm_td/`: Core C module for QMK (`sm_td.c`, `sm_td.h`, `qmk_module.json`).
- `tests/unit/`: Python `unittest` suites — compile and call the C library via `ctypes` and `clang` (see `tests/unit/sm_td_bindings.py`). Feature folders (e.g., `caps_word_enable/`) contain `layout.c` and `test.py`.
- `tests/integration/`: QMK-native googletest suites — compile `sm_td.c` against real `qmk_firmware` checkout. Suites in `suites/smtd_*/`, scripts `fetch.sh`/`run.sh`.
- `tests/run_tests.py`: Python test runner (discovers tests in `tests/unit/`).
- `docs/`: User docs and customization guides.
- `keyboards/`: Example QMK integration assets.

## Architecture
- Entrypoint: `process_smtd(...)` interprets tap vs. hold and schedules deferred actions.
- Hooks: implement `on_smtd_action(...)` and configure `SMTD_GLOBAL_*` macros for timing/features.
- Tests: Python drives C via `ctypes`; timeouts simulated with `prolong()` in key helpers (event-driven) or with the virtual clock `smtd.wait(ms)` (time-driven, required for dynamic timeouts).
- Key output ("pipeline taps", since #23 fix): `SMTD_TAP_16`/`SMTD_REGISTER_16`/`SMTD_UNREGISTER_16` call `smtd_tap_code16`/`smtd_register_code16`/`smtd_unregister_code16`. When the sent key equals the keymap keycode at the pressed position (and `use_cl != false`), the key goes through `smtd_emulate_key` -> full `process_record()` pipeline (Caps Word, combos, etc. see it); otherwise a manual `process_caps_word()` shim runs before a direct `tap_code16` send. Controlled by `SMTD_GLOBAL_PIPELINE_TAPS` (default true) and per-key `SMTD_FEATURE_PIPELINE_TAPS`.
- `smtd_executing_state` tracks the state whose action is running (set in `smtd_execute_action`); `state->emulated_register` pairs an emulated register with an emulated release even if the layer changed in between. `smtd_emulate_key` saves/restores `smtd_bypass` because it can be called while bypass is already set.
- Leader support (#29 fix): `process_leader` runs after `process_record_user` in the quantum chain, so it only sees taps that re-enter via `process_record()`. Native-keycode taps already do (pipeline path), so leader worked for them since the #23 pipeline-taps change. Custom/derived-keycode taps take the direct `tap_code16` fallback and bypassed leader — `smtd_tap_code16` now calls `smtd_leader_consume(key)` (guarded by `#ifdef LEADER_ENABLE`) before the direct send, mirroring `process_leader` (`get_tap_keycode` unless `LEADER_KEY_STRICT_KEY_PROCESSING`, `leader_sequence_add`, `LEADER_PER_KEY_TIMING` timer reset, `leader_end` on full buffer). Only the tap path is covered, not holds during a sequence. Regression suite: `tests/integration/suites/smtd_leader/` (native + custom keycode, both must yield the leader marker).

## Build, Test, and Development Commands
- `just test` — all tests (python + qmk native)
- `just test python` — python unit tests only
- `just test qmk` — all qmk native suites against default QMK version
- `just test qmk full` — specific suite (smtd_full); prefix `smtd_` is auto-added
- `just test qmk 0.32.16` — all suites against a specific QMK version
- `just test qmk 0.32.16 full` — specific version + suite
- `just test qmk-matrix` — all pinned QMK versions × all suites
- `just configure` — set QMK checkout directory (saved in `.env`, gitignored)
- `just fetch-qmk` / `just fetch-qmk 0.32.16` — download QMK without running tests
- Direct python: `python3 tests/run_tests.py`, `python3 -m unittest tests.unit.caps_word_enable.test`
- Direct qmk native: `sh tests/integration/run.sh 0.33.5 smtd_full` (respects `SMTD_QMK_DIR` env var)
Notes:
- First `just test qmk` prompts for QMK checkout directory and saves it to `.env`.
- Python tests auto-compile a per-suite shared library with `clang -shared -fPIC` (macOS: `.dylib`, Linux: `.so`). No manual `cmake` step is needed.
- QMK native tests need `make` and a C++ toolchain (QMK's googletest harness).
- Prereqs: Python 3.10+, `clang` on PATH, `just` (https://github.com/casey/just).

## Coding Style & Naming Conventions
- C11 for `sm_td`; Python for tests.
- C: snake_case for functions (`process_smtd`), ALL_CAPS for macros (`SMTD_*`), K&R braces, 4-space indents (prefer spaces in new code).
- Headers: keep QMK includes behind guards; use `static` for internal symbols; prefix public API with `smtd_`/`SMTD_`.

## Testing Guidelines
- Framework: Python `unittest` driving the C library via `ctypes`.
- Structure: each feature folder has `layout.c` and `test_*.py`; use helpers in `tests/unit/sm_td_assertions.py`.
- Naming: discovered by `test*.py`; keep scenario names descriptive.
- Determinism: seed randomness in tests that sample (e.g., `random.seed(…)` in `setUp`) to stabilize failures.
- Known baseline failures (pre-existing): `test_stirred_long_mod_smtd_press2_fixed` in `caps_word_enable` and `complex_layout` (4 FAILs total) - prolong/defer assertion issue, unrelated to recent work.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stasmarkin/sm_td](https://github.com/stasmarkin/sm_td) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
