---
trigger: always_on
description: > Spec-Driven Development framework for Data Engineering on Claude Code
---

# AgentSpec Development

> Spec-Driven Development framework for Data Engineering on Claude Code

---

## Project Context

**What is AgentSpec?** A Claude Code plugin that provides structured AI-assisted development through a 5-phase SDD workflow, specialized for data engineering with 58 agents, 30 commands, 23 KB domains, and 2 skills.

**Current Status:** v3.0.0 — Claude Code plugin distribution complete. Linear is the project tracker (source of truth).

---

## Repository Structure

```text
agentspec/
├── .claude/                 # Claude Code integration
│   ├── agents/              # 58 specialized agents
│   │   ├── architect/       # 8 system-level design agents
│   │   ├── cloud/           # 10 AWS, GCP, cloud services, CI/CD
│   │   ├── platform/        # 6 Microsoft Fabric specialists
│   │   ├── python/          # 6 Python dev, code quality, prompts
│   │   ├── test/            # 3 testing, data quality, contracts
│   │   ├── data-engineering/ # 15 DE implementation specialists
│   │   ├── dev/             # 4 developer tools & productivity
│   │   └── workflow/        # 6 SDD phase agents
│   │
│   ├── commands/            # 30 slash commands
│   │   ├── workflow/        # SDD commands (7)
│   │   ├── data-engineering/ # DE commands (8)
│   │   ├── core/            # Utility commands (5)
│   │   ├── knowledge/       # KB commands (1)
│   │   ├── review/          # Review commands (1)
│   │   └── visual-explainer/ # Visual documentation commands (8)
│   │
│   ├── skills/              # Reusable capability packs
│   │   ├── visual-explainer/ # HTML page generation (templates, references, scripts)
│   │   └── excalidraw-diagram/ # Excalidraw JSON generation
│   │
│   ├── sdd/                 # SDD framework
│   │   ├── architecture/    # WORKFLOW_CONTRACTS.yaml, ARCHITECTURE.md
│   │   ├── templates/       # 5 document templates (DE-aware)
│   │   ├── features/        # Active development
│   │   ├── reports/         # Build reports
│   │   └── archive/         # Shipped features
│   │
│   └── kb/                  # Knowledge Base (23 domains)
│       ├── _templates/      # 7 KB domain templates
│       ├── _index.yaml      # Domain registry
│       ├── dbt/             # dbt patterns and concepts
│       ├── spark/           # PySpark, Spark SQL
│       ├── sql-patterns/    # SQL best practices
│       ├── airflow/         # DAG patterns
│       ├── streaming/       # Flink, Kafka, CDC
│       ├── data-modeling/   # Star schema, Data Vault, SCD
│       ├── data-quality/    # GE, Soda, observability
│       ├── lakehouse/       # Iceberg, Delta, catalogs
│       ├── cloud-platforms/ # Snowflake, Databricks, BigQuery
│       ├── ai-data-engineering/ # RAG, vector DBs, features
│       ├── modern-stack/    # DuckDB, Polars, SQLMesh
│       ├── aws/             # Lambda, S3, Glue, SAM
│       ├── gcp/             # Cloud Run, Pub/Sub, BigQuery
│       ├── microsoft-fabric/ # Lakehouse, Warehouse, Pipelines
│       ├── lakeflow/        # Databricks Lakeflow (DLT)
│       ├── medallion/       # Bronze/Silver/Gold architecture
│       ├── prompt-engineering/ # Chain-of-thought, extraction
│       ├── genai/           # Multi-agent systems, guardrails
│       ├── pydantic/        # Validation, LLM output schemas
│       ├── python/          # Python patterns and idioms
│       ├── testing/         # pytest, fixtures, CI testing
│       └── terraform/       # IaC modules, state, workspaces
│
├── docs/                    # Documentation
│   ├── getting-started/     # Installation and first pipeline
│   ├── concepts/            # SDD pillars through DE lens
│   ├── tutorials/           # dbt, star schema, Spark, streaming tutorials
│   └── reference/           # Full catalog: agents, commands, KB domains
│
├── plugin/                  # Generated plugin (built by build-plugin.sh)
│   ├── .claude-plugin/      # Plugin manifest + marketplace config
│   ├── agents/              # Copied + path-rewritten agents
│   ├── commands/            # Copied + path-rewritten commands
│   ├── skills/              # 5 skills (3 from .claude/ + 2 plugin-only)
│   ├── kb/                  # Copied KB domains
│   ├── sdd/                 # Templates + architecture (no features/reports/archive)
│   ├── hooks/               # SessionStart workspace init
│   └── scripts/             # init-workspace.sh
│
├── plugin-extras/           # Plugin-only content (merged into plugin/ by build)
│   ├── skills/              # sdd-workflow, data-engineering-guide
│   ├── hooks/               # hooks.json
│   └── scripts/             # init-workspace.sh
│
├── build-plugin.sh          # Builds plugin/ from .claude/ (invokes scripts/generate-agent-router.py)
├── scripts/
│   └── generate-agent-router.py  # Regenerates agent-router SKILL.md + routing.json from agent frontmatter
├── CHANGELOG.md             # Version history
├── CONTRIBUTING.md          # Contribution guide
├── SECURITY.md              # Security policy
└── README.md                # Project overview
```

---

## Development Workflow

Use AgentSpec's own SDD workflow to develop AgentSpec:

```bash
# Explore an enhancement idea
/brainstorm "Add Judge layer for spec validation"

# Capture requirements
/define JUDGE_LAYER

# Design the architecture

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [luanmorenommaciel/agentspec](https://github.com/luanmorenommaciel/agentspec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
