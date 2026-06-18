---
trigger: always_on
description: `greenfield_database` is the source repository for the Charles River synthetic accounting analytics teaching environment. The primary artifact is the Docusaurus documentation site; the SQLite/Excel/support-workbook/CSV outputs and generated report assets exist to support that site and classroom use.
---

# AGENTS.md

## Project Overview

`greenfield_database` is the source repository for the Charles River synthetic accounting analytics teaching environment. The primary artifact is the Docusaurus documentation site; the SQLite/Excel/support-workbook/CSV outputs and generated report assets exist to support that site and classroom use.

The default package is a student-facing, anomaly-enriched teaching build. Separate clean, validation, and performance configs exist for reconciliation work and smaller local runs.

## Repository Map

- `docs/`: authored student/instructor docs. Files are Markdown with YAML front matter, but many pages use MDX imports/components inside `.md`.
- `queries/`: curated SQL packs for financial, managerial, audit, and case work. These files are also served directly by Docusaurus as static assets.
- `config/`: source-of-truth settings and catalogs. Key files are `settings*.yaml`, `anomaly_profile.yaml`, `accounts.csv`, `report_catalog.yaml`, and `report_pack_catalog.yaml`.
- `src/generator_dataset/`: Python dataset generator, exporters, settings, validations, and schema registry. `schema.py` is the canonical table/column definition source.
- `src/components/`, `src/pages/`, `src/theme/`, `src/css/`: Docusaurus UI code and custom MDX components.
- `src/generated/queryManifest.js`, `src/generated/reportManifest.js`, `src/generated/reportDocCollections.js`, `src/generated/reportPackManifest.js`: script-generated site data. Regenerate; do not hand-edit.
- `src/generated/queryDocCollections.js`: despite the folder name, no generator script is present in this repo. Treat it as curated source that must be kept in sync manually with query/docs changes.
- `scripts/`: Node scripts for branding/manifests/report-asset prep plus `scripts/generate-site-reports.py` for report export from an existing SQLite file.
- `static/`: site assets. `static/CNAME`, `static/img/favicon.svg`, and `static/img/site-social-card.svg` are generated from branding settings. `static/reports/` holds generated report preview/download assets and is ignored.
- `outputs/`: local generated datasets, reports, and logs. Ignored.
- `tests/`: pytest coverage for generator behavior, SQL semantics, report exports, docs structure, and teaching-flow invariants.
- `.github/workflows/deploy-docs.yml`: GitHub Pages build/deploy workflow.

## Setup And Commands

- Local prerequisites:
  - Python is required for dataset generation/tests. CI uses Python `3.13`.
  - Node `>=20` is required locally. CI uses Node `24`.
- Python environment:
  ```bash
  python -m venv .venv
  # activate the environment for your shell
  pip install -r requirements.txt
  ```
- For all agent-driven Python commands, use the repo-local virtual environment instead of the system interpreter. Prefer invoking the interpreter directly, for example:
  ```bash
  .venv\Scripts\python.exe -m pip install -r requirements.txt
  .venv\Scripts\python.exe -m pytest -q
  .venv\Scripts\python.exe -B -m compileall -q src tests
  ```
  On non-Windows shells, use `.venv/bin/python`.
- Node dependencies:
  ```bash
  npm ci
  ```
- Docusaurus development server:
  ```bash
  npm run start
  ```
- Production site build:
  ```bash
  npm run build
  ```
- Serve the built site:
  ```bash
  npm run serve
  ```
- Clear Docusaurus cache:
  ```bash
  npm run clear
  ```
- Individual manifest/branding/report-asset commands:
  ```bash
  npm run generate-site-branding
  npm run generate-query-manifest
  npm run generate-report-manifest
  npm run generate-report-pack-manifest
  npm run prepare-report-assets
  npm run validate-report-assets
  ```
- Dataset generation:
  ```bash
  .venv\Scripts\python.exe generate_dataset.py
  .venv\Scripts\python.exe generate_dataset.py config/settings_validation.yaml core
  ```
- Report asset generation from an existing SQLite file:
  ```bash
  .venv\Scripts\python.exe scripts/generate-site-reports.py --config config/settings.yaml --sqlite-path outputs/CharlesRiver.sqlite --report-output-dir static/reports
  ```
- Tests and Python syntax sanity:
  ```bash
  .venv\Scripts\python.exe -m pytest -q
  .venv\Scripts\python.exe -B -m compileall -q src tests
  ```
- Any agent- or tool-driven test invocation must use a timeout that matches the scope. Use at least one hour (`3600000` ms) for small targeted slices, and at least two hours (`7200000` ms) for broad multi-module runs or full-suite validation.
- Package deploy command:
  ```bash
  npm run deploy
  ```

Notes:

- `npm run start`, `npm run build`, and `npm run serve` already run the pre-hooks that generate branding/manifests and prepare/validate report assets.
- If `static/reports/` is missing, `npm run start` and `npm run build` may try to download the published SQLite asset using `published_sqlite_url` in `config/settings.yaml` or `REPORTS_SQLITE_URL`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mmcodesso/CharlesRiver_Database](https://github.com/mmcodesso/CharlesRiver_Database) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
