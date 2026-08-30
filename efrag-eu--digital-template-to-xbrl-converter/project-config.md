---
trigger: always_on
description: generates viewers/xBRL-JSON via `arelle.api.Session`; `taxonomy_info` is the build-time side that
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

EFRAG's converter that turns a filled-in VSME Excel "Digital Template" into a validated Inline XBRL
report (plus report package, viewer and xBRL-JSON). Mapping is driven by Excel **named ranges** whose
names match the local name of the corresponding taxonomy concept. Ships as both a CLI (`scripts/`) and
a Flask web app.

## Environment and commands

Two virtualenvs exist in the checkout. `.venv-py314` is the default for all work; `.venv-py311` exists
only for checking the 3.11 baseline (`requires-python = ">=3.11"`, which is what CI runs). Never invoke
a bare `python`.

```powershell
.venv-py314/Scripts/python -m pip install -e ".[dev]"    # editable install with dev extras
```

### Tests

The three test trees are run separately in CI and are best run separately locally too.

```powershell
.venv-py314/Scripts/python -m pytest tests/unitTests
.venv-py314/Scripts/python -m pytest tests/integrationTests
.venv-py314/Scripts/python -m pytest tests/webappTests

# single test / single case
.venv-py314/Scripts/python -m pytest tests/unitTests/xlsx_template_reader/test_ranges.py::test_name
```

- Tests marked `@pytest.mark.slow` (notably full Arelle validation in
  `tests/integrationTests/test_expectedFactCounts.py`) are **skipped by default**. Enable with
  `--run-slow` or `FORCE_RUN=1`. They run automatically on `main` and PRs to `main`.
- After any change to the conversion pipeline, run `tests/integrationTests/test_expectedFactCounts.py`
  — it pins per-template fact counts and catches silent fact loss.
- `tests/unitTests/xlsx_template_reader/test_fact_creator_characterization.py` snapshots every fact
  (concept, value, aspects, footnotes) from the 1.2.0 and 1.3.0 samples. Regenerate deliberately, only
  when a change to the output is intended:
  `.venv-py314/Scripts/python tests/unitTests/xlsx_template_reader/test_fact_creator_characterization.py`
- `tests/unitTests/test_import_cycles.py` fails on any new circular import inside `mireport`.

### Lint and types

CI runs ruff only, but mypy config is strict-ish for `src/` (`disallow_untyped_defs`, tests excluded).

```powershell
.venv-py314/Scripts/python -m ruff check .
.venv-py314/Scripts/python -m ruff format --check .
.venv-py314/Scripts/python -m mypy src
```

### Running the converter

```powershell
# Excel -> Inline XBRL (add --viewer / --json; output path may be a file or a directory)
.venv-py314/Scripts/python scripts/parse-and-ixbrl.py example.xlsx output.html

# dump named ranges from a workbook (debugging what the reader sees)
.venv-py314/Scripts/python scripts/parse-and-dump.py example.xlsx

# validate an existing report / build a viewer for it
.venv-py314/Scripts/python scripts/check-report.py report.html

# web app (auto-reload)
.venv-py314/Scripts/python -m flask --app digital_converter_webapp run --debug
```

`--skip-validation` on `parse-and-ixbrl.py` is a development-only shortcut. Do not propose it as a way
to make things faster in anything that matters — Arelle validation is the point of the tool.

### Regenerating taxonomy data

Taxonomy metadata is **pre-baked JSON** in `src/mireport/data/taxonomies/`; nothing at runtime reads
`.xsd`/linkbases. The JSON is produced by Arelle from taxonomy package zips (which are not in this
repo):

```powershell
.venv-py314/Scripts/python scripts/update-taxonomy.py --entry-point <URL> src/mireport/data/taxonomies/vsme-YYYY-MM-DD.json path/to/*.zip
```

`scripts/dump-taxonomy.py` dumps concept/presentation info (including to xlsx) from the baked JSON.

### Front-end assets

`src/digital_converter_webapp/static/style.css` is a **generated, committed** Tailwind build. Edit
`src/digital_converter_webapp/templates/source.css`, then regenerate (requires `npm install`):

```powershell
npx @tailwindcss/cli -i .\src\digital_converter_webapp\templates\source.css -o .\src\digital_converter_webapp\static\style.css --minify
npx eslint .
```

## Architecture

Two packages under `src/`: `mireport` (all the conversion logic, no Flask) and
`digital_converter_webapp` (thin Flask UI over it). The CLI scripts and the web app are two front ends
driving the same pipeline.

### The pipeline

```
.xlsx  ──XlsxProcessor──▶  InlineReport (facts + aspects)
       ──Jinja templates──▶  HTML in the aoix template dialect
       ──ixbrltemplates.Parser──▶  Inline XBRL
       ──▶ report package (.zip)  ──Arelle──▶  validation messages / viewer / xBRL-JSON
```

`InlineReport.getInlineReport()` renders `mireport/report/inline_report_templates/` and then hands the
HTML to `aoix` (`ixbrltemplates`), which rewrites the template markup into real `ix:` tagging. This is
why report templates contain aoix-specific attributes rather than hand-written Inline XBRL, and why
default aspects (entity, currency, periods, numeric transforms) are passed to the template as an
`aoix` dict rather than applied per fact.

### Taxonomy layer (`mireport/taxonomy.py`)

`loadBuiltInTaxonomyJSON()` must run before `getTaxonomy(entryPoint)` — every entry point (CLI, web
app, test conftest) does this first. Taxonomies live in a module-level registry keyed by entry-point

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EFRAG-EU/Digital-Template-to-XBRL-Converter](https://github.com/EFRAG-EU/Digital-Template-to-XBRL-Converter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
