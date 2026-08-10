---
trigger: always_on
description: DocMeld is a lightweight PDF-to-agent-ready knowledge pipeline. Three-stage architecture: Bronze (PDF → JSON elements) → Silver (JSON → page-by-page JSONL) → Gold (JSONL → AI-enriched metadata).
---

# DocMeld Development Guidelines

## Project Overview

DocMeld is a lightweight PDF-to-agent-ready knowledge pipeline. Three-stage architecture: Bronze (PDF → JSON elements) → Silver (JSON → page-by-page JSONL) → Gold (JSONL → AI-enriched metadata).

## Active Technologies
- Python 3.9+ + PyMuPDF (fitz), pymupdf4llm, pydantic, langchain-deepseek (all existing) (002-paper-batch-categorize)
- Filesystem (JSON files) — no database (002-paper-batch-categorize)

- Python 3.9+
- PyMuPDF (fitz), pymupdf4llm, Docling (optional)
- pandas, openpyxl, pydantic, python-dotenv, langchain-deepseek
- Testing: pytest, pytest-cov, ruff, black, mypy

## Project Structure

```text
docmeld/
├── docmeld/           # Source code
│   ├── bronze/        # PDF → JSON extraction
│   │   └── backends/  # ParserBackend implementations (pymupdf, docling)
│   ├── silver/        # JSON → JSONL page conversion
│   ├── gold/          # JSONL → AI-enriched metadata
│   ├── utils/         # Shared utilities
│   ├── parser.py      # Main DocMeldParser class
│   └── cli.py         # CLI entry point
├── tests/             # Unit, integration, contract tests
├── pyproject.toml
└── venv/
specs/                 # Feature specifications (speckit managed)
references/00-core/    # Reference docs, examples, use cases
```

## Commands

```bash
cd /Users/frank/A/DocMeld/docmeld
source venv/bin/activate
pytest tests/ -v                              # Run all tests
pytest tests/ -v --cov=docmeld               # With coverage
ruff check docmeld/                           # Lint
black --check docmeld/                        # Format check
mypy docmeld/                                 # Type check
```

## Code Style

- Python 3.9+ minimum, use `from __future__ import annotations`
- Black formatting (line-length=100)
- Ruff linting, Mypy strict mode
- TDD: write tests first per project constitution

## Git Branching Strategy

```
main                          ← stable, tested, merge target
├── 001-mvp-pdf-pipeline      ← completed MVP
├── 002-use-case-*            ← next use case feature branch
├── 003-use-case-*            ← ...
└── NNN-feature-name          ← each feature gets a numbered branch
```

- `main` is the integration branch. All feature branches merge to main after passing tests.
- Each use case / feature gets its own numbered branch via `/speckit.specify`.
- Never push directly to main. Always merge from a feature branch.

## Harness Coding Loop

This project uses a structured harness loop for autonomous development. Each use case from `references/00-core/use-cases.md` is processed through this pipeline:

### The Loop

```
┌─────────────────────────────────────────────────────┐
│              USE CASE BACKLOG                        │
│  references/00-core/use-cases.md                    │
│  Pick next unprocessed use case                     │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│  STEP 1: SPECIFY                                    │
│  /speckit.specify "NNN-use-case-short-name"         │
│  → Creates branch, writes specs/NNN-*/spec.md       │
│  → Validates spec quality checklist                 │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│  STEP 2: PLAN                                       │
│  /speckit.plan                                      │
│  → Generates plan.md with architecture + file map   │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│  STEP 3: TASKS                                      │
│  /speckit.tasks                                     │
│  → Generates tasks.md with ordered, testable tasks  │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│  STEP 4: IMPLEMENT                                  │
│  /speckit.implement                                 │
│  → TDD: write tests first, then implement           │
│  → Mark tasks [x] as completed                      │
│  → Phase-by-phase execution                         │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│  STEP 5: TEST & EVALUATE                            │
│  pytest tests/ -v --cov=docmeld                     │
│  ruff check docmeld/ && black --check docmeld/      │
│  → All tests must pass                              │
│  → Coverage must not regress                        │
│  → If failures: fix and re-run (do not skip)        │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│  STEP 6: MERGE TO MAIN                              │
│  git checkout main                                  │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentii-ai/DocMeld](https://github.com/agentii-ai/DocMeld) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
