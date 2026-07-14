---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Three related tools in one Python project ("fetlib"):

1. **MOSFET datasheet parser & parametric search** — scrape manufacturer sites, download PDFs, extract specs, build a CSV ranked by estimated DC-DC power loss.
2. **Power-loss modelling for synchronous buck** — gate-charge-curve based HS/LS loss estimate (more accurate than `Rds_on * Qg` FoM).
3. **Inductor (powder core) design & loss** — `maglib/`, sendust/Micrometals-style materials with DC-bias and core-loss models.

The README is mostly accurate for usage; this file captures the architecture and runtime contract that's not obvious from reading individual files.

## Running things

There is no `pyproject.toml`/`setup.py` — this is run as scripts from the repo root with `python3.9` and `requirements.txt` in a venv.

```bash
python3 main.py --config-file apps/proj/buck.yaml    # full pipeline driven by a YAML
python3 main.py                                       # uses DcDcLoadParams.default()
python3 discover_parts.py                             # only the parts-discovery + download phase
python3 datasheet.py <command> <pdf>                  # single-file utility: open|ascii|parse|read-sheet-debug|power|html|html-pm|rasterize
python3 power_loss_calc.py                            # plot loss curves for hard-coded MPPT designs in apps/mppts/
```

YAML projects live in `apps/proj/*.yaml` (`buck.yaml`, `fugu*.yaml`, `mppt*.yaml`). Schema is consumed in `main.py:main_yaml` → `RunArgs` / `DcdcArgs` / `ControlFetArgs` / `SyncFetArgs` / `InductorArgs`; each load point becomes a `DcDcLoadParams`. Currently only `topology: buck` is accepted and exactly one load point per run.

Tests use pytest (no `pytest.ini` / `conftest.py`):

```bash
pytest test/unit                  # focused unit tests (text norm, pdf tree, mosfet specs, etc.)
pytest test/tests.py              # broader, heavier integration-style tests
pytest test/unit/test_mosfet_specs.py::test_name   # single test
```

Many tests under `test/` are loose scripts (`benchmark.py`, `pdf2table.py`, `plumber.py`, …) — not pytest, run directly with `python` when needed.

## External tooling (must be installed; not pure-Python)

The datasheet pipeline shells out to / drives several non-Python tools — missing them silently degrades parsing. From the README:

- **Tabula** (Java) — `tabula-py` plus the Tabula app GUI used as a browser-style table extractor. `dslib/pdf/tabular.py` launches Tabula via file locks `.tabula_browser_{1..5}.lock` at the repo root, capped by `tabula_browser_concurrency = 5`. On macOS Apple Silicon use Zulu JDK.
- **Ghostscript ≥ 9.55**, **poppler-utils**, **qpdf**, **sips**, **CUPS-PDF**, **FontForge** (`dslib/pdf/fonts.py` shells `fontforge_bin`), **Tesseract** (used via `ocrmypdf`).
- **Chromium via `pyppeteer`** for anti-bot datasheet downloads — uses a persistent profile at `dslib/chromium-user-data-dir/`. **Chromium's built-in PDF viewer must be disabled** (`"plugins": {"always_open_pdf_externally": true}` in `Default/Preferences`) — otherwise PDF downloads can't be captured.

## Architecture: discovery → fetch → parse → model → CSV

The pipeline in `main.py:run` is a linear flow; understanding it requires reading several modules together.

### 1. Discovery — `dslib/discovery/`

Per-manufacturer scrapers (`infineon.py`, `ti.py`, `toshiba.py`, `st.py`, `onsemi.py`, `vishay.py`, `nxp.py` (nexperia), `ao.py` (alpha&omega), `tw.py` (taiwansemi), `huayi.py`, `qorvo.py`, `epc.py` (GaN), `lcsc.py`, `digikey.py`, `china.py`) each return `List[DiscoveredPart]`. `discover_parts.py:discover_mosfets` runs them all (mostly `async`) and merges with `unique_parts`.

`unique_parts` deduplicates by `(mfr, normalized_mpn)`. Infineon-specific suffix stripping (`AKMA1`, `AKSA1`, `XKSA1`, `XKMA1`) is applied. **Digikey rows are treated as untrustworthy** — if a duplicate already exists, a digikey-only entry is dropped rather than merged (comment: "digikey data is often wrong"). Otherwise `.specs.update(part.specs)` merges fields from later sources into earlier ones.

Digikey input is CSVs under `parts-lists/digikey/*.csv` (downloaded manually from the Digikey parametric search, 500 results max per CSV). LCSC inputs are HTML DOM dumps under `parts-lists/lcsc/`.

Pre-selection by Vds/Id happens in `DcDcLoadParams.select_mosfets(parts, max_parallel=…)` — this is what filters down to candidates worth downloading datasheets for.

### 2. Datasheet fetch — `dslib/fetch.py`

`fetch_datasheet(url, dest, mfr=, mpn=)` handles manufacturer-specific redirects, anti-bot challenges, and PDF-preview pages via pyppeteer. Files land at `datasheets/<mfr>/<mpn>.pdf` (computed by `DiscoveredPart.get_ds_path()`). The `datasheets/` directory is gitignored and ships as a separate repo: `https://github.com/open-pe/fet-datasheets`.

### 3. PDF parsing — `dslib/pdf/`

Parsing is **deliberately multi-strategy with a priority order** (`README.md` "Field priority"); LLMs were tried and rejected as non-deterministic. Order of preference per field:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fl4p/fetlib](https://github.com/fl4p/fetlib) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
