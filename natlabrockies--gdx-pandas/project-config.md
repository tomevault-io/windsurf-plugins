---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this package does

`gdxpds` translates between GDX (GAMS Data eXchange) files and pandas DataFrames. GDX is the binary file format used by [GAMS](https://www.gams.com/), a mathematical optimization modeling system. Two entry points:

- High-level functions: `to_dataframes()`, `to_dataframe()`, `list_symbols()`, `get_data_types()`, `to_gdx()` — exposed at package top level.
- Object-oriented API: `GdxFile` and `GdxSymbol` in [src/gdxpds/gdx.py](src/gdxpds/gdx.py) for programmatic, lazy access.

## Runtime dependency on GAMS

This package **cannot function without a GAMS installation** — there is no mock layer. The SWIG-bound GDX bindings talk to the GAMS shared library found at runtime, and are imported **lazily** (on the first GDX operation), so `import gdxpds` itself does not need a binding. Two equivalent binding sources are supported via `try/except` imports (inside the engine modules and the lazy-load helpers, not at package import):

- **Modern (recommended):** `from gams.core import gdx as gdxcc` — shipped inside `gamsapi`, which the user installs version-matched to their GAMS install (`pip install gamsapi[transfer]==xx.y.z`). Not a base dependency of gdxpds.
- **Legacy:** `import gdxcc` — the standalone PyPI package. Available via the `[legacy]` extra (`pip install gdxpds[legacy]`). Older but the SWIG C ABI is stable enough that it still works.

Other runtime notes:

- GAMS lookup order is implemented by `GamsDirFinder` in [src/gdxpds/tools.py](src/gdxpds/tools.py): `GAMS_DIR` env var → `GAMSDIR` env var → `where gams` / `which gams` → walk default install location (`C:\GAMS` on Windows; picks highest version). The Windows walk handles both the modern `C:\GAMS\<version>\` layout and the legacy `C:\GAMS\win64\<version>\` layout by looking for `gams.exe` to identify a GAMS root.
- `GAMS_DIR` remains mandatory at runtime even with pip-installed bindings, because the GDX shared library lives in the GAMS install directory, not in the wheel. The recommended pattern is one venv per GAMS install with `$Env:GAMS_DIR` pinned via `Activate.ps1` — see [dev/README.md](dev/README.md).
- **`import gdxpds` works with no binding installed.** The `gdxcc.GMS_*` type codes are hardcoded in the `GamsDataType`/`GamsVariableType`/`GamsEquationType`/`GamsValueType` enums (with `tests/test_imports.py::test_gms_constants_match_gdxcc` verifying them against the live binding when present), and the bindings load on the first GDX op. So `gdxpds info` / `gdxpds test` can diagnose the "no bindings installed" environment.

If tests fail with "cannot load gdxcc" or "no `_gdxcc` module," it's a GAMS environment problem (missing `GAMS_DIR`, missing bindings, or version skew between `gamsapi` and the GAMS install), not a code bug.

## Common commands

PowerShell on Windows. Always activate the venv first:

```powershell
.venv\Scripts\Activate.ps1
```

Install for development:

```powershell
pip install -e .[test]
```

Run the full test suite:

```powershell
pytest tests
```

Run a single test file or test:

```powershell
pytest tests/test_read.py
pytest tests/test_read.py::test_name
```

Keep test output files after a run (useful when debugging round-trip failures):

```powershell
pytest tests --no-clean-up
```

The custom `--no-clean-up` flag and the shared test fixtures (`base_dir`, `run_dir`, `manage_rundir`, `roundtrip_one_gdx`) are defined in [tests/conftest.py](tests/conftest.py).

The installed `gdxpds` CLI exposes three subcommands:

```powershell
gdxpds --version    # terse version line
gdxpds info         # environment report (Python, bindings, GAMS_DIR + source, load status)
gdxpds test         # end-to-end install verification against the local GAMS
```

`gdxpds info` is also the Python function [gdxpds.info()](src/gdxpds/__init__.py) — it returns the report as a `str` and is contracted to never raise.

Verify a fresh install end-to-end (intended for end users; ships with the base package, no `[test]` extra needed):

```powershell
gdxpds test
```

Source lives in [src/gdxpds/cli/main.py](src/gdxpds/cli/main.py); the embedded
sample GDX is at `src/gdxpds/_verify_install/sample.gdx`, regenerable via
[dev/build_verify_install_sample.py](dev/build_verify_install_sample.py).

Build the docs (Sphinx, MyST-flavored markdown sources):

```powershell
pip install -e .[docs]   # or .[dev] for tests + docs
cd doc
.\make.bat html
```

Output is in `doc/build/html/`. Hand-authored docs are `.md` (parsed by MyST). The API page is generated automatically by `sphinx.ext.autosummary` with `:recursive:` — see [doc/source/api.md](doc/source/api.md) and the templates in [doc/source/_templates/autosummary/](doc/source/_templates/autosummary/). Full release / docs publish workflow — GitHub Actions on Release-published events — is in [dev/README.md](dev/README.md).

## Architecture notes

Things that aren't obvious from one file:

- **Lazy loading.** `GdxFile` (a `MutableSequence` of `GdxSymbol`) defaults to `lazy_load=True`. Symbol data is only pulled from the GDX file when `.dataframe` is accessed. Iterating symbol metadata is cheap; touching dataframes is not.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NatLabRockies/gdx-pandas](https://github.com/NatLabRockies/gdx-pandas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
