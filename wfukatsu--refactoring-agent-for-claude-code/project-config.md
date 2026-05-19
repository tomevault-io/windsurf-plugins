---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Claude Code agent system for analyzing legacy systems and planning microservices refactoring. The main orchestrator (`/refactor-system`) coordinates 18 specialized sub-agents through a sequential pipeline, producing Markdown analysis reports, API specifications, and a RyuGraph knowledge graph database.

## Quick Start

```bash
# Full pipeline (all phases in sequence)
/refactor-system ./SampleCode

# Individual phases (run after /analyze-system produces reports/01_analysis/)
/analyze-system ./SampleCode          # Phase 1: ubiquitous language, actors, domain-code mapping
/evaluate-mmi ./SampleCode            # Phase 2: MMI evaluation (requires Phase 1)
/map-domains ./SampleCode             # Phase 3: bounded contexts, context map
/design-microservices ./SampleCode    # Phase 4: target architecture, migration plan
/design-api ./SampleCode              # Phase 4.5: REST/GraphQL/gRPC/AsyncAPI specs
/design-scalardb ./SampleCode         # Phase 5: distributed transaction design
/create-domain-story --domain=Order   # Phase 6: interactive domain storytelling
/estimate-cost ./reports              # Phase 7: infrastructure + license cost

# Knowledge graph (parallel with main pipeline)
/build-graph ./SampleCode             # Build RyuGraph from Phase 1 outputs
/query-graph "注文に関連するクラス"     # Natural language or Cypher query
/visualize-graph ./reports/graph      # Mermaid/DOT/HTML output

# Utilities
/compile-report                       # Markdown → single HTML report
/render-mermaid ./reports             # Mermaid → PNG + SVG via mmdc
/fix-mermaid ./reports                # Auto-fix Mermaid syntax errors
/scalardb-sizing-estimator            # Interactive Pod/K8s/DB sizing & cost

# Orchestrator options
/refactor-system ./src --output=./custom-output/
/refactor-system ./src --domain=Order,Customer
/refactor-system ./src --analyze-only
/refactor-system ./src --skip-mmi
/refactor-system ./src --skip-stories
```

## Python Utilities

```bash
pip install ryugraph pandas markdown pymdown-extensions

python scripts/parse_analysis.py --input-dir ./reports/01_analysis --output-dir ./reports/graph/data
python scripts/build_graph.py --data-dir ./reports/graph/data --db-path ./knowledge.ryugraph
python scripts/query_graph.py --db-path ./knowledge.ryugraph --interactive
python scripts/visualize_graph.py --data-dir ./reports/graph/data --output-dir ./reports/graph/visualizations
python scripts/compile_report.py --input-dir ./reports --output ./reports/00_summary/full-report.html
```

## Architecture

### Prompt Pipeline Pattern

The system uses a "prompt-as-code" architecture: each skill is an LLM instruction document that Claude Code follows step-by-step. The orchestrator invokes skills sequentially via `Skill()` calls, with each skill reading prior phase outputs and writing its own files immediately.

```
User → /refactor-system ./SampleCode
  → LLM reads .claude/skills/refactor-system/SKILL.md
  → Invokes Skill(analyze-system) → writes reports/01_analysis/*.md
  → Invokes Skill(evaluate-mmi)   → writes reports/02_evaluation/*.md
  → Invokes Skill(map-domains)    → writes reports/03_design/domain-analysis.md, context-map.md
  → Invokes Skill(design-microservices) → writes reports/03_design/target-architecture.md, etc.
  → Invokes Skill(design-api)     → writes reports/03_design/api-*.md + api-specifications/
  → Invokes Skill(design-scalardb) → writes reports/03_design/scalardb-*.md
  → Invokes Skill(create-domain-story) → writes reports/04_stories/
  → Invokes Skill(estimate-cost)  → writes reports/05_estimate/
  → Invokes Skill(fix-mermaid)    → validates all Mermaid diagrams
  → Writes reports/00_summary/executive-summary.md

Parallel: /build-graph → parse CSVs → knowledge.ryugraph/
```

**Critical convention**: Skills must write output files immediately upon completing each step ("最後にまとめて出力しない"). Do not buffer all output until the end.

### Skill System

Skills live in `.claude/skills/{skill-name}/SKILL.md` with YAML frontmatter (`name`, `description`, `user_invocable`). Each contains step-by-step LLM instructions, tool usage guidance, and output format specifications.

Commands in `.claude/commands/{skill-name}-cmd.md` are the user-facing entry points (with `-cmd` suffix). They mirror the skill content but are formatted as Claude Code slash commands with `description` and `argument-hint` frontmatter.

The template at `.claude/templates/output-structure.md` defines the canonical file dependency graph and required sections for each output file — it acts as a contract between skills.

### Dual Invocation

- **As skill**: `Skill(analyze-system)` — used by the orchestrator or programmatic invocation
- **As command**: `/analyze-system-cmd ./src` — user-typed slash command in Claude Code
- Both resolve to the same SKILL.md instructions

### Sample Target System

`SampleCode/` contains **"Scalar Auditor for BOX"** — an enterprise audit application used as the canonical test target:

| Directory | Stack | Description |
|-----------|-------|-------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wfukatsu/refactoring-agent-for-claude-code](https://github.com/wfukatsu/refactoring-agent-for-claude-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
