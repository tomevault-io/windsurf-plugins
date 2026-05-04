---
trigger: always_on
description: This repository is a Stata empirical-research workflow template. Codex should preserve the existing Stata pipeline behavior while using this file as the primary project instruction source.
---

# AGENTS.md - Codex Operating Guide

This repository is a Stata empirical-research workflow template. Codex should preserve the existing Stata pipeline behavior while using this file as the primary project instruction source.

## Project Purpose

- Maintain a reproducible Stata workflow for empirical economics research.
- Keep `dofiles/00_master.do` as the single end-to-end pipeline entry point.
- Keep raw data out of version control.
- Produce audit-friendly tables, figures, logs, and Quarto reports.
- Preserve the existing Claude Code assets under `.claude/` as reference material; do not remove them unless the user explicitly asks.

## Repository Map

- `dofiles/00_master.do`: canonical pipeline orchestrator.
- `dofiles/01_clean/`: raw data to cleaned data.
- `dofiles/02_construct/`: variable construction and samples.
- `dofiles/03_analysis/`: estimation, robustness, event studies, IV, DiD, etc.
- `dofiles/04_output/`: table and figure assembly.
- `dofiles/_utils/`: reusable Stata helpers.
- `data/raw/`: raw datasets, gitignored.
- `data/derived/`: intermediate datasets, gitignored.
- `logs/`: Stata logs, gitignored.
- `output/tables/`: committed publication/audit tables.
- `output/figures/`: committed figures.
- `reports/`: Quarto reports.
- `scripts/`: wrappers and quality tooling.
- `explorations/`: self-contained sandbox analyses and teaching demos.
- `templates/`: reusable project templates.
- `.claude/`: legacy Claude Code agents, skills, rules, and hooks. Treat as reference.

## Non-Negotiable Rules

- No numerical research claim without a source in `logs/*.log` or `output/tables/*`.
- Do not fabricate or infer coefficients, standard errors, p-values, sample sizes, or summary statistics.
- Do not commit or expose raw or derived datasets from `data/raw/` or `data/derived/`.
- Do not weaken `.gitignore` data-protection rules without explicit user confirmation.
- Use relative paths in Stata code. Avoid hardcoded machine-specific paths.
- Every substantive `.do` file should have `version`, `set more off`, `set varabbrev off`, logging, and a clear header.
- Write new or modified Stata do-file comments in Chinese unless the user requests another language.
- Keep reports downstream of pipeline outputs. Reports should consume `output/`, not become the primary analysis source.
- Do not edit protected files casually: `dofiles/00_master.do`, `.gitignore`, and bibliography files if added later.

## Commands

Run a single do-file:

```bash
bash scripts/run_stata.sh dofiles/03_analysis/main_regression.do
```

Run the full pipeline:

```bash
bash scripts/run_pipeline.sh
```

Render the report:

```bash
quarto render reports/analysis_report.qmd
```

Score an artifact:

```bash
python scripts/quality_score.py dofiles/03_analysis/main_regression.do
python scripts/quality_score.py reports/analysis_report.qmd
python scripts/quality_score.py scripts/check_data_safety.py
```

Check staged files for data leaks:

```bash
python scripts/check_data_safety.py --staged $(git diff --cached --name-only)
```

## Stata Conventions

- Local Stata is Stata 15: `C:\Program Files (x86)\Stata15\Stata-64.exe`.
- Pin Stata do-files to `version 15` unless the user explicitly changes the project standard.
- Pin the Stata version at the top of each `.do` file.
- Use one project-wide seed unless a task has a documented reason to do otherwise.
- Open and close logs inside runnable do-files.
- Store estimation results with `estimates store` or `est store` when table assembly depends on them.
- Export figures with native Stata `graph export` as both `.pdf` and `.png`; do not keep `.gph` as a committed artifact.
- When generating a figure, always create both PDF and PNG outputs unless Stata itself cannot run.
- Use a muted Stata-style graph design by default: white or very light gray background, subtle horizontal gridlines only, solid blue marks or bars using RGB `"49 145 255"` / HEX `#3191FF`, no glossy or gradient-like effects, no strong contrast edges, Arial or default Stata Sans fonts, normal-weight dark blue-gray titles, modest axis/tick label sizes, and Stata-default-like proportions with enough whitespace.
- Prefer `esttab` outputs as `.tex` plus `.csv` for auditability.
- Cluster standard errors at the most defensible aggregate level and document the choice.

## Local Environment

- Python is Miniconda, not the Microsoft Store Python.
- Miniconda root: `C:\ProgramData\Miniconda3`.
- Conda executable: `C:\ProgramData\Miniconda3\Scripts\conda.exe`.
- Python executable: `C:\ProgramData\Miniconda3\python.exe`.
- Stata executable: `C:\Program Files (x86)\Stata15\Stata-64.exe`.
- Prefer explicit executable paths if `python`, `conda`, or `stata` are not found on `PATH`.

## Log Verification

Before stating a numerical result:

1. Find the current log or output table.
2. Verify the value appears in that artifact.
3. Cite the artifact path and, when practical, the surrounding context or line.
4. If no current artifact exists, say that the do-file must be run first.

Use this response pattern when blocked:

```text

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maxwell2732/codex-stata-for-economists](https://github.com/maxwell2732/codex-stata-for-economists) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
