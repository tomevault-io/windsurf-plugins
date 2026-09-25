---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Claude Code skill (`/persona`) inspired by TinyTroupe that builds market-specific persona panels and pressure-tests product concepts before fieldwork. It generates diverse personas, simulates independent concept test responses via `claude -p` subprocesses (one per persona), analyzes results, and produces executive reports.

## Common Commands

```bash
# Run all tests (150 unit tests, no external dependencies needed)
python -m pytest tests/ -q

# Run a single test file
python -m pytest tests/test_simulate_survey.py -q

# Run a single test
python -m pytest tests/test_simulate_survey.py::ValidateResponseTests::test_accepts_canonical_nested_concept_response -q

# Run survey simulation (requires claude CLI authenticated)
python scripts/simulate_survey.py --config demo/running-shoes/concept-test/config.json

# Dry-run (preview prompts without executing)
python scripts/simulate_survey.py --config demo/running-shoes/concept-test/config.json --dry-run

# Run with analysis + LLM report
python scripts/simulate_survey.py --config demo/running-shoes/concept-test/config.json --analyze --report-llm

# Run with a different model (aliases or full IDs; e.g. Claude Fable 5)
python scripts/simulate_survey.py --config ... --model fable --fallback-model sonnet

# Run analysis only on existing results
python scripts/analyze_results.py --input outputs/.../results.json --survey-type concept-test

# Install analysis dependencies (only needed for analyze_results.py)
pip install pandas matplotlib seaborn
```

## Architecture

### 3-Step Workflow (maps to TinyTroupe)

| Step | Skill | TinyTroupe equivalent |
|------|-------|----------------------|
| 1. Build Panel | Panel Builder generates diverse personas | TinyPersonFactory |
| 2. Run Concept Test | Simulation Engine runs agent-separated interviews | TinyWorld |
| 3. Review Findings | Analysis Pipeline produces report + charts | Extract & Analyze |

### Three-Script Engine

All runtime logic lives in `scripts/`. There is no `src/` package — scripts import each other via `sys.path` manipulation.

1. **`simulate_survey.py`** (1500 lines) — The main entry point. Loads config JSON, loads personas from `panel_dir`, builds per-persona prompts from templates, fans out parallel `claude -p` subprocesses, validates JSON responses, retries failures, and saves `results.json` + `run_metadata.json`.

2. **`analyze_results.py`** (1400 lines) — Reads `results.json`, normalizes response structures, generates CSV exports, cross-tabulations, charts (matplotlib/seaborn), and a markdown report. Can use an LLM backend for narrative report generation or fall back to deterministic Python templates.

3. **`llm_backends.py`** (~600 lines) — Backend abstraction for `claude-cli` and `codex-cli`. Handles CLI command construction (`_build_claude_print_command()`), defensive envelope parsing (`_parse_claude_envelope()` — `structured_output` preferred, fenced-text extraction as fallback), CLI capability detection (`detect_claude_capabilities()` greps `claude --help` once per process so new flags degrade gracefully on older CLIs), async/sync subprocess communication, and model resolution. The `"auto"` backend infers which CLI is available via environment markers.

4. **`validate_panel.py`** (663 lines) — Panel quality gate. Runs 11 checks against a generated panel: count vs. requested, name uniqueness, segment balance, occupation/surname diversity, geo spread, age spread, gender distribution, Big Five cosine similarity (flags pairs ≥ 0.98), and slot-plan adherence. Returns structured JSON via `--json`; exits non-zero on any hard fail. Called automatically after panel generation; can also be run standalone for diagnosis.

### Data Flow

```
Config JSON (e.g., demo/running-shoes/concept-test/config.json)
    ↓ specifies survey_type, panel_dir, variables, backend
Persona JSONs (personas/{survey-id}/*.json + manifest.json)
    ↓ loaded via manifest.json's persona_files list
Profile Extraction (full ~300-line persona → ~40-line simulation profile)
    ↓ topic-relevant fields selected, verbose fields dropped
Per-Persona Prompt (simulation-prompt.md + template + profile + variables)
    ↓ each persona gets independent claude -p subprocess
Response Validation (validate_response checks required keys per survey type)
    ↓ retries up to 3 times on validation failure
results.json + run_metadata.json (outputs/{date}/{time}/{survey_type}/)
    ↓ analyze_results.py
report.md, results.csv, summary.json, charts (same output dir)
```

### Config JSON Structure

```json
{
  "survey_type": "concept-test",
  "panel_dir": "personas/{survey-id}",
  "topic": "Research topic",
  "variables": { "category": "...", "concepts": "..." },
  "output_dir": "outputs/...",
  "backend": "claude-cli|codex-cli|auto",
  "model": "sonnet",
  "max_concurrency": 5,

  "report_model": "fable",
  "fallback_model": "sonnet",
  "effort": "medium",
  "max_budget_usd_per_call": 0.5,
  "structured_output": true,
  "isolation": true
}
```

The keys below the blank line are optional (added in 0.2.0) and default to

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [takechanman1228/claude-persona](https://github.com/takechanman1228/claude-persona) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
