---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Environment

All Python scripts must run under `~/venv`. The scripts self-reinvoke under the venv automatically, but for direct execution:

```bash
~/venv/bin/python generate_reports.py <metric_csv>
~/venv/bin/python parse_metrics_csv.py <metric_csv> [brand_name]
~/venv/bin/python build_heytruffle_reports.py <metric_csv> [--brands "..."] [--trend-csvs "..."]
~/venv/bin/python upload_to_drive.py [--month January_2026]
```

`build_heytruffle_reports.py` also shells out to `node generate_report.js` — `npm install` must have been run once (see `package.json`).

## Two report pipelines

There are currently **two generators** in this repo, both driven by the same Metabase CSV export:

1. **RestoHost pipeline** (original, `generate_reports.py`) — Python-only, fills `template.pptx` directly via `python-pptx`. See "Running Reports" below.
2. **heytruffle pipeline** (new, `build_heytruffle_reports.py`) — the rebrand target. Design is decoupled from data: a Node/`pptxgenjs` generator draws the "Monthly Impact Report" layout from scratch and reads all values from a per-brand JSON file. See "heytruffle Pipeline" below.

The heytruffle pipeline is intended to replace the RestoHost one once all clients are onboarded (`clients_config.json` populated + curated insights written for each). Until then, treat the RestoHost pipeline as the still-in-use default for any brand not yet configured in `clients_config.json`.

## Running Reports (RestoHost pipeline)

```bash
# All brands from a single CSV
~/venv/bin/python generate_reports.py /path/to/exported.csv

# Specific brands only
~/venv/bin/python generate_reports.py /path/to/exported.csv --brands "Felino,KYU"

# With trend chart (ordered oldest→newest CSVs; only applies to brands in --brands)
~/venv/bin/python generate_reports.py /path/to/exported.csv \
  --brands "Rreal Tacos" \
  --trend-csvs "nov.csv,dec.csv,jan.csv"

# Debug a single brand's metrics without generating PPTX
~/venv/bin/python parse_metrics_csv.py /path/to/exported.csv "Brand Name"
```

## Running Reports (heytruffle pipeline)

```bash
# One brand, with a 3-month trend for the "calls handled" chart
~/venv/bin/python build_heytruffle_reports.py /path/to/may.csv \
  --trend-csvs "mar.csv,apr.csv,may.csv" --brands "Aplos Mediterranean"

# All brands present in clients_config.json (default when --brands is omitted)
~/venv/bin/python build_heytruffle_reports.py /path/to/exported.csv
```

Per-brand steps performed by `build_heytruffle_reports.py`:
1. Load that brand's rows from the month CSV (skips the brand if absent).
2. Look up the brand in `clients_config.json` (skips the brand if no entry — see below).
3. `report_auto.compute_auto()` — reuses `parse_metrics_csv.stream_brand_metrics()` for `recovered_count`/`roi_hours`, and separately computes `assisted_revenue` from `smsByCategory` "guest action" categories (pickup, waitlist, catering, reservation, delivery, private events): `revenue = guest_actions × 80% conversion × guests × avg_ticket`. It also computes the "Main Insights" block (`AI Resolution Rate` and `Calls Handled`, each compared against the previous month if `--trend-csvs` has a prior-month entry) and a `has_trend_history` flag — `false` when fewer than 3 months of trend data exist, in which case `generate_report.js` renders a "Solved by AI" resolution chart instead of the (otherwise sparse) calls-handled bar chart.
4. Load `data/curated/<Brand>_<YYYY-MM>.json` (hand-written `listen` block only — two real call reasons; everything else in the report is auto-computed). If missing, `TODO` placeholders are used and a warning is printed — the report will still generate but needs a manual pass before sending.
5. Merge `client` (from `clients_config.json`) + `period` + `auto` + `curated` into `data/generated/<Brand>_<YYYY-MM>.json`.
6. Run `node generate_report.js <data.json> <out.pptx>` to render the PPTX (`pptx/<Brand>_Report_<Month><Year>.pptx`).

**`clients_config.json`** (gitignored — real client data, never committed) holds per-brand `name`, `tagline`, `locations_line`, `logo`, `guests` (avg party size), and `avg_ticket`. `clients_config.example.json` (committed) shows the expected shape — copy it to `clients_config.json` and fill in real values for each brand. `avg_ticket` currently defaults to a flat **$25** per brand (Google Places-based estimation is not wired up yet, see `HEYTRUFFLE_REBRAND.md`); override per-brand once real data exists — `Aplos Mediterranean` already has a validated value ($20) and should not be changed without re-checking against real data.

**`data/curated/`** and **`data/generated/`** are gitignored — they contain real client metrics and hand-written call-highlight data and must never be committed.

## Architecture

The pipeline is single-pass and sequential per brand:

**`generate_reports.py`** — orchestrator. Loads the CSV once via `load_all_brand_rows()`, then for each brand: aggregates metrics → generates chart → builds PPTX → accumulates summary.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [twais-resto/reports-generation](https://github.com/twais-resto/reports-generation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
