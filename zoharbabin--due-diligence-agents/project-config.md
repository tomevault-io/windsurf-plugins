---
trigger: always_on
description: Python application for forensic M&A due diligence. Analyzes contract data rooms across 4 domains (Legal, Finance, Commercial, ProductTech) using 8 AI agents under a 35-step pipeline with 5 blocking gates. Produces a detailed cross-domain HTML report + 14-sheet Excel report with structured findings, citations, and audit trail. The reports provide granular analysis that deal teams use as the basis for their own deliverables — IC memos, advisor reports, negotiation checklists, or integration plans.
---

# Due Diligence Agent SDK — Claude Code Instructions

## Project Overview

Python application for forensic M&A due diligence. Analyzes contract data rooms across 4 domains (Legal, Finance, Commercial, ProductTech) using 8 AI agents under a 35-step pipeline with 5 blocking gates. Produces a detailed cross-domain HTML report + 14-sheet Excel report with structured findings, citations, and audit trail. The reports provide granular analysis that deal teams use as the basis for their own deliverables — IC memos, advisor reports, negotiation checklists, or integration plans.

**Package**: `dd-agents` on [PyPI](https://pypi.org/project/dd-agents/) / `dd_agents` under `src/dd_agents/`
**Version**: see `pyproject.toml` (bump version there before tagging a release)
**SDK**: `claude-agent-sdk>=0.1.56` (Python 3.12+, tested on 3.12 and 3.13)
**Spec**: 24 plan docs in `docs/plan/`. Start with `docs/plan/PLAN.md`.

## Commands

```bash
# Install (end users)
pip install dd-agents[pdf]

# Install (development)
pip install -e ".[dev,pdf]"

# Test (run after EVERY change)
pytest tests/unit/ -x -q                    # Unit tests (~3,300, fast, no API)
pytest tests/integration/ -x -q             # Integration tests (mock agents)
pytest tests/e2e/ -x -q                     # E2E tests (requires API, expensive)

# Type check
mypy src/ --strict

# Lint
ruff check src/ tests/
ruff format src/ tests/ --check

# All quality gates at once
pytest tests/unit/ -x -q && mypy src/ --strict && ruff check src/ tests/

# Build package locally
python -m build && twine check dist/*

# Run the pipeline
dd-agents run path/to/deal-config.json
```

## Architecture

- **Orchestrator** (`orchestrator/engine.py`): 35 async steps as methods on `PipelineEngine`. State machine with checkpoint/resume.
- **Agents** (`agents/`): 4 specialists (Legal, Finance, Commercial, ProductTech) + Judge + Executive Synthesis + Red Flag Scanner + Acquirer Intelligence. Spawned via `claude-agent-sdk`.
- **Persistence**: Three tiers — PERMANENT (never wiped), VERSIONED (archived per run), FRESH (rebuilt each run).
- **Hooks** (`hooks/`): PreToolUse hooks return flat `{"decision": "block"|"allow", "reason": "..."}`. Stop hooks use SDK format `{"continue_": bool, "stopReason": "..."}`. Never nest under `hookSpecificOutput`. PreToolUse chain: (1) bash_guard, (2) path_guard, (3) file_size_guard, (4) aggregate_file_guard, (5) finding_schema_guard — validates finding JSON structure on Write to `findings/{agent}/*.json`, blocking wrong field names like `evidence` instead of `citations`. Stop hook: check_coverage + check_manifest (relaxed — allows stop when all subject JSONs are written; orchestrator backfills manifests post-session).
- **Models** (`models/`): Pydantic v2 for all schemas. `model_json_schema()` for structured outputs. Note: some BaseModel subclasses live outside `models/` by design — agent output schemas (`agents/*.py`), report templates (`reporting/templates.py`), query models (`query/*.py`), and internal helpers (`orchestrator/batch_scheduler.py`, `validation/pre_merge.py`, `extraction/coordinates.py`) are co-located with their consumers for cohesion.
- **Validation** (`validation/`): 6-layer numerical audit, 31 substantive DoD checks (content-validated, not file-existence). Fail-closed — validation failures block the pipeline.
- **Knowledge** (`knowledge/`): Deal Knowledge Base — persistent knowledge layer that compounds across runs. 12 modules: `base.py` (article CRUD + atomic writes), `articles.py` (Pydantic models), `compiler.py` (findings → articles), `graph.py` (NetworkX knowledge graph), `chronicle.py` (append-only JSONL timeline), `lineage.py` (SHA-256 finding fingerprinting), `health.py` (7-category integrity checks), `prompt_enrichment.py` (agent context builder), `filing.py` (file-back to data room), `search_context.py` (search enrichment interface), `index.py` (auto-maintained JSON index), `_utils.py` (shared helpers). Compiled automatically in step 32 unless `--no-knowledge` is passed.

## Code Style

- Python 3.12+, strict mypy, ruff for lint/format
- Line length: 120 characters
- Pydantic v2 models with Field descriptions for every field
- Async functions for pipeline steps
- All JSON schemas validated via Pydantic `model_validate()`
- `subject_safe_name`: lowercase, strip legal suffixes (Inc/Corp/LLC/Ltd), replace special chars with `_`, collapse underscores. Example: "Smith & Partners, Inc." → `smith_partners`
- Reporting terminology: internal code uses "subject"; HTML/Excel report outputs use "Entity" for external-facing content
- Batch naming is 1-based: `batch_1`, `batch_2` (never `batch_0`)

## CI/CD

Two GitHub Actions workflows in `.github/workflows/`:

### CI (`ci.yml`) — runs on every push/PR to `main`

```
Stage 1 (parallel):   Lint & Format, Type Check (mypy --strict)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zoharbabin/due-diligence-agents](https://github.com/zoharbabin/due-diligence-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
