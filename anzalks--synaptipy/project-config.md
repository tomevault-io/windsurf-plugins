---
trigger: always_on
description: manages view ranges explicitly via `_reset_view()`.
---

# Synaptipy — GitHub Copilot Instructions

## Language and framework
- Python 3.10–3.12, PySide6 (Qt6), pyqtgraph, NumPy, SciPy
- Tests use pytest + pytest-qt; CI runs on ubuntu/windows/macos × Python 3.10/3.11/3.12
- Linting: `flake8 src/ tests/` with `.flake8` config (max-line-length=120, max-complexity=10)
- Formatting: `black` (line-length=120, target-version=py310) and `isort` (profile=black)
- CI enforces `black --check`, `isort --check`, and `flake8` — PRs that fail any of these are rejected

## Code style
- Follow PEP 8; max line length 120 characters
- All code must be formatted with `black` and imports sorted with `isort`
- All public functions and classes must have docstrings
- Use type hints throughout
- Keep function complexity ≤ 10 (flake8 C901)
- **Typography**: Strictly use standard hyphens (`-`). Never use em dashes (`—`) or en dashes (`–`) in code, documentation, or changelogs.

## Analysis registry pattern
- New analysis functions are registered with `@AnalysisRegistry.register(name=..., ui_params=[...], plots=[...])`
- `ui_params` entries drive the GUI automatically; use `visible_when: {"param": "...", "value": "..."}` for conditional visibility
- Wrapper functions must return a plain `dict`; private keys (starting with `_`) are hidden from the results table

### Registry import rule — DO NOT import only registry.py
To populate the `AnalysisRegistry`, **import the full package**
`import synaptipy.core.analysis` (which triggers `__init__.py` → `from . import
basic_features`, etc.).  **Never** rely on
`from synaptipy.core.analysis.registry import AnalysisRegistry` alone — that
only imports the class and does NOT execute the analysis sub-modules' decorators.
This was the root cause of the Windows bug where the Analyser tab showed 0 tabs
while macOS showed 15 (on macOS the batch engine happened to be imported earlier
via a different path, masking the issue).

### Editable install must point to the active workspace
`pip install -e .` stores the editable project location.  If the repo is cloned
to a new directory, the old editable link still points to the previous path.
Run `pip install -e .` from the new workspace to update.  Symptom: modules
visible on disk (`capacitance.py`, `optogenetics.py`, `train_dynamics.py`) throw
`ModuleNotFoundError` because Python resolves the package from the stale path.

### Preprocessing reset must propagate globally
`BaseAnalysisTab._handle_preprocessing_reset()` is connected to
`PreprocessingWidget.preprocessing_reset_requested`.  When fired it must:
1. Clear `_active_preprocessing_settings`, `_preprocessed_data`, and `pipeline`
2. Call `preprocessing_widget.reset_ui()` to reset combo boxes to "None"
3. Walk up to the parent `AnalyserTab` and call `set_global_preprocessing(None)`
   so **all** sibling tabs also reset
4. Clear `SessionManager().preprocessing_settings`
5. Re-plot with raw data

`apply_global_preprocessing(None)` (called on sibling tabs) must also call
`preprocessing_widget.reset_ui()` so every tab's UI visually reflects the reset.

## CI / test rules — DO NOT VIOLATE

### PySide6 version constraint — DO NOT WIDEN
`requirements.txt`, `pyproject.toml`, and `environment.yml` all pin
`pyside6==6.7.3` (exact).  **Do not remove or loosen this pin.**

PySide6 6.10.x changed the internal signal-connection machinery so that
deferred ViewBox geometry callbacks re-queue themselves *during* `connect()`
calls inside `PlotItem.__init__`.  Because the canvas reuses a single
`GraphicsLayoutWidget` across many test invocations, stale post-`widget.clear()`
events that are still in the queue when the next `addPlot()` runs dereference
already-freed C++ pointers → access violation (Windows) / segfault (macOS).
This was diagnosed across CI runs 22418950288 – 22420935486 on all platforms.
The constraint must stay until pyqtgraph ships a fix or we switch to creating
a fresh `GraphicsLayoutWidget` per rebuild cycle.

### Local macOS exit codes are misleading
`pytest_sessionfinish` calls `os._exit()` (macOS/Linux) or
`kernel32.TerminateProcess()` (Windows) in offscreen mode; the macOS process
always prints `Abort trap: 6` even when all tests passed.  **Never judge a
local macOS run by the shell exit code.**  Check the pytest output lines
(`N passed`, zero `FAILED`):
```bash
conda run -n synaptipy python -m pytest tests/ 2>&1 | grep -c PASSED
conda run -n synaptipy python -m pytest tests/ 2>&1 | grep "FAILED\|ERROR "
```

### Windows pytest_sessionfinish uses TerminateProcess — DO NOT CHANGE
`pytest_sessionfinish` calls `kernel32.TerminateProcess(ctypes.c_void_p(-1), int(exitstatus))`
on Windows.  **Do not replace with `os._exit()`, `RtlExitUserProcess`, or any
other function.**

- `os._exit()` calls `ExitProcess()` which fires `DLL_PROCESS_DETACH` on all
  loaded DLLs (including Qt) → access violation on freed Qt C++ objects.
- `ntdll.RtlExitUserProcess()` still calls `LdrShutdownProcess()` which also
  fires `DLL_PROCESS_DETACH` → same crash.  Confirmed via faulthandler traceback
  in CI run 22433271920: crash at `conftest.py::pytest_sessionfinish` line 76.
- `kernel32.TerminateProcess()` is the only API that truly bypasses
  `DLL_PROCESS_DETACH` (MSDN: "does not run the DLL entry function with
  DLL_PROCESS_DETACH").

`argtypes` and `ctypes.c_void_p(-1)` are mandatory:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anzalks/synaptipy](https://github.com/anzalks/synaptipy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-28 -->
