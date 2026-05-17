---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**SkillFlow** is an agent skill retrieval system that enables AI agents to discover and execute skills from online sources. The project has four main components:

1. **Skill Crawler** (`skill_crawler/`): Crawls and downloads ~36K agent skills from online marketplaces (SkillsMP) into `data/skills/`
2. **SkillFlow Core** (`skill_flow/`): A multi-stage semantic skill retrieval engine (FAISS vector search → cross-encoder reranking → optional second reranker → LLM selection) over ~36K agent skills
3. **Evaluation Framework** (`benchmark/`): A Harbor-based benchmarking system to evaluate agents with and without skill augmentation using SkillsBench and Terminal-Bench
4. **Analysis** (`analysis/`): Retrieval comparisons, statistical analysis, and paper table/figure generation
5. **Paper** (`paper/`): LaTeX source for the SkillFlow research paper, with Overleaf synchronization workflow

## Commands

```bash
# Install dependencies
uv sync

# Build FAISS index (one-time, requires data/skills/)
uv run python -m skill_flow.cli build-index

# Search the index
uv run python -m skill_flow.cli search --query "write unit tests for FastAPI"

# Search with cross-encoder reranking (Stage 2)
uv run python -m skill_flow.cli search --query "write unit tests for FastAPI" --rerank

# Run auto-chained evaluation (all 4 stages, default: skill_flow/config/default_eval.json)
uv run python -m skill_flow.cli eval

# Run eval with a specific config (e.g., retriever-only via default.json)
uv run python -m skill_flow.cli eval --config skill_flow/config/default.json

# Run benchmark evaluation CLI
uv run python -m benchmark.scripts.cli run --config benchmark/config/default.json

# Run tests with coverage
uv run pytest tests/ -v

# Run retriever comparison experiment
uv run python -m skill_flow.cli experiment --config skill_flow/config/experiments/retriever-comparison.json --max-tasks 2

# Run reranker comparison experiment
uv run python -m skill_flow.cli experiment --config skill_flow/config/experiments/reranker-comparison.json --max-tasks 2

# Compare two retriever eval reports (per-task win/loss analysis)
uv run python -m analysis.comparison.compare_retrievers REPORT_A REPORT_B --k 10 --top-n 10

# Regenerate all paper tables and figures from analysis data
bash analysis/results/generate-paper-assets.sh            # both
bash analysis/results/generate-paper-assets.sh --tables   # tables only
bash analysis/results/generate-paper-assets.sh --figures  # figures only

# Push paper/ to Overleaf (one-way sync)
bash paper/scripts/push-overleaf.sh

# Push paper/ to Overleaf with history reset (when histories diverge)
bash paper/scripts/push-overleaf.sh --reset

# Crawl all skill sources
uv run python -m skill_crawler crawl

# Crawl specific source
uv run python -m skill_crawler crawl --source skillsmp

# Dry run (list without downloading)
uv run python -m skill_crawler crawl --dry-run

# Search SkillsMP skills
uv run python -m skill_crawler search "PDF editing"

# Check crawler status
uv run python -m skill_crawler status

# Validate downloaded skills
uv run python -m skill_crawler validate ./data/skills
```

## Architecture

```
skill-flow/
├── skill_crawler/              # Skill crawler (data acquisition)
│   ├── __init__.py
│   ├── cli.py                  # Typer CLI (crawl, search, status, validate, dedupe)
│   ├── config.py               # Pydantic settings (env vars + config.json)
│   ├── config/
│   │   └── config.json         # Default crawler configuration
│   ├── models/                 # Skill model, sync state
│   ├── crawlers/               # Source-specific crawlers (SkillsMP API/scraper)
│   ├── downloaders/            # GitHub archive/SKILL.md downloader
│   ├── storage/                # Save/index/validate/deduplicate skills
│   └── utils/                  # HTTP client, progress output
│
├── skill_flow/                 # SkillFlow core library
│   ├── __init__.py
│   ├── cli.py                  # CLI entry point (build-index, search, eval, experiment)
│   ├── models/                 # Domain models
│   │   ├── __init__.py         # SkillRecord (Pydantic, frozen)
│   │   └── core.py             # SkillFlow facade (retriever + reranker + deep_reranker + selector composition)
│   ├── config/                 # Configuration
│   │   ├── __init__.py         # Pydantic config models (Config, SystemConfig, IndexConfig, ModelsConfig, RetrieverConfig, RerankerConfig, DeepRerankerConfig, SelectorConfig, QueryGenConfig, RetrieverVariant, RetrieverExperimentConfig, RerankerVariant, RerankerExperimentConfig, *EvalSettings)
│   │   ├── default.json        # Default config (system/index/models hierarchy)
│   │   ├── default_eval.json           # Eval default config (all 4 stages enabled, auto-chained)
│   │   └── experiments/        # Config presets for evaluation experiments
│   │       ├── eval-deep-reranker.json # Config preset for deep reranker evaluation
│   │       ├── eval-selector.json  # Config preset for selector-only evaluation
│   │       ├── retriever-comparison.json      # Multi-retriever experiment (dense + BM25)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IBPA/skill-flow](https://github.com/IBPA/skill-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
