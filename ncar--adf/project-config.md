---
trigger: always_on
description: **Status: draft.** This file tells an AI agent (e.g. Claude) how to review a pull request to
---

# AGENTS.md — reviewing pull requests to the ADF

**Status: draft.** This file tells an AI agent (e.g. Claude) how to review a pull request to
the AMWG Diagnostics Framework (ADF). It is about *review*, not about running the ADF — user
and developer instructions live on the [wiki](https://github.com/NCAR/ADF/wiki).

Upstream repo is `NCAR/ADF`; PRs target `main`.

---

## 1. What the ADF is, in one paragraph

The ADF is a Python framework that produces standard climatological comparisons of CAM runs
(case vs. case, or case vs. obs/reanalysis). A run is driven by a single YAML config file
(`./run_adf_diag config_cam_baseline_example.yaml`) and proceeds in fixed stages: create time
series → create climatologies → regrid/vertically interpolate → analysis → plotting → build a
static website. The framework (`lib/`) owns configuration, data access, and the website; the
science (`scripts/`) is contributed largely by CAM users and is plugged in by name from the
config file.

**Key consequence for review:** almost everything under `scripts/` is dynamically imported by
name and called with a single argument. There is no static wiring to catch a broken interface,
and CI does not lint these files (see §3). Interface and runtime errors here surface only in a
real ADF run, so the review has to do that job.

## 2. Repository layout

| Path | What it is |
| --- | --- |
| `run_adf_diag` | Top-level executable driver; orders the stages. |
| `lib/adf_base.py` | `AdfBase`: debug log, `debug_log()`, `end_diag_fail()`, `AdfError`. |
| `lib/adf_config.py` | `AdfConfig`: YAML read, `${var}` reference expansion, `read_config_var()`. |
| `lib/adf_info.py` | `AdfInfo`: derived run info and the properties/getters scripts read (`diag_var_list`, `plot_location`, `climo_yrs`, `case_nicknames`, `hist_string`, `get_basic_info()`, `get_cam_info()`, `get_baseline_info()`, …). |
| `lib/adf_dataset.py` | `AdfData`: **the** data-access layer — `load_climo_da`, `load_regrid_da`, `load_reference_*`, `get_*_file`, unit converters. |
| `lib/adf_derive.py` | Variable derivation (`check_derive`, `derive_variable`). |
| `lib/adf_obs.py` | Observation-file bookkeeping. |
| `lib/adf_web.py` | `AdfWeb`: `add_website_data()` plus HTML generation from `lib/website_templates/`. |
| `lib/adf_diag.py` | `AdfDiag`: the stage methods and the dynamic script caller. |
| `lib/plotting_functions.py`, `lib/plotting_utils.py`, `lib/adf_utils.py` | Shared plotting/utility helpers used by `scripts/`. |
| `lib/adf_variable_defaults.yaml` | Per-variable plotting/obs/vector/website/derivation defaults. `..._era5-1deg.yaml` is the 1° ERA5 variant. |
| `scripts/averaging/`, `scripts/regridding/`, `scripts/analysis/`, `scripts/plotting/` | Pluggable stage scripts, selected by the config file's `time_averaging_scripts`, `regridding_scripts`, `analysis_scripts`, `plotting_scripts`. |
| `lib/test/unit_tests/` | pytest suite (currently `adf_base`, `adf_config` only). |
| `lib/test/pylintrc` | pylint config used by CI. |
| `.github/workflows/`, `.github/scripts/` | CI (see §3). |
| `config_*.yaml` | Example configs kept in sync with the code. |

## 3. What CI actually checks — and what it does not

Three workflows run on every PR:

1. **`ADF_unit_tests.yaml`** — `pytest lib/test/unit_tests` on Python 3.9–3.13. Only PyYAML and
   pytest are installed, so anything importing xarray/matplotlib cannot be unit-tested as written.
2. **`ADF_linting.yaml`** — `.github/scripts/pr_mod_file_tests.py` runs pylint at
   **threshold 9.5** with `lib/test/pylintrc`, but only on modified files in this hard-coded
   `testable_files` set:
   `lib/adf_base.py`, `lib/adf_config.py`, `lib/adf_info.py`, `lib/adf_obs.py`,
   `lib/adf_web.py`, `lib/adf_diag.py`.
3. **`ADF_pre-commit.yaml`** — `pre-commit run -a`; the only hook configured is `check-yaml`.

**Un-linted, un-tested by CI:** everything in `scripts/`, plus `lib/adf_dataset.py`,
`lib/adf_derive.py`, `lib/adf_utils.py`, `lib/plotting_functions.py`, `lib/plotting_utils.py`.
Weight the review accordingly — a PR touching only `scripts/plotting/` gets a green checkmark
from a suite that never imported it.

Reproduce the checks locally:

```bash
pytest lib/test/unit_tests
pylint --rcfile=lib/test/pylintrc lib/adf_diag.py          # any file in testable_files
pre-commit run -a
python -c "import yaml,sys; yaml.safe_load(open(sys.argv[1]))" config_cam_baseline_example.yaml
```

A useful extra pass that CI will not do — syntax/import sanity on changed scripts:

```bash
python -m py_compile $(git diff --name-only main...HEAD -- '*.py')
```

## 4. Contracts to verify

These are the invariants that make the plug-in architecture work. Check every one that the
diff touches.

### 4.1 The stage-script interface

For a script `scripts/<stage>/<name>.py` listed in the config:

- The **module basename and the entry-point function name must match** (`global_latlon_map.py`
  must define `global_latlon_map`). `AdfDiag.__function_caller()` does
  `importlib.import_module(name)` then `getattr(module, name)`, and calls
  `end_diag_fail()` if the name is absent.
- The entry point is called as `func(adfobj)` — or `func(adfobj, **kwargs)` when the config

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NCAR/ADF](https://github.com/NCAR/ADF) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
