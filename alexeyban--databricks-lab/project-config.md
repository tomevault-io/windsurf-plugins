---
trigger: always_on
description: **Before making any changes to this repository:**
---

# AGENTS.md

## Workflow for Changes

**Before making any changes to this repository:**

1. Create a new branch for your changes
2. Make your changes on that branch
3. Create a pull request
4. **Wait for human approval** before merging

```bash
# Example workflow
git checkout -b feature/your-feature-name
# ... make your changes ...
git add .
git commit -m "Description of changes"
git push -u origin feature/your-feature-name
# Then create PR via GitHub UI or: gh pr create --title "..." --body "..."
```

Do NOT commit directly to main/master.

## Purpose
This repository is a Databricks CDC lakehouse lab with:
- local Docker infrastructure
- Python automation and generator scripts
- Databricks notebook utilities
- a dbt project in `cdc_gold/`
Use this file as the repo-specific guide for coding agents.

## Rule Files Checked
- `.cursor/rules/`: not present
- `.cursorrules`: not present
- `.github/copilot-instructions.md`: not present
No extra Cursor or Copilot rules were found, so follow this file and nearby code.

## Repo Layout
- `runtime/`: Databricks helper modules and notebook normalization
- `generators/`: PostgreSQL data mutation generators
- `Agents/`: local specialist agent definitions and role prompts
- `skills/`: local skill activators and repo-specific operational workflows
- `pipeline_configs/silver/`: per-table JSON configs for generic Silver pipelines
- `dq_queries/silver/`: stored SQL data quality checks executed after Silver updates
- `skills/docker-databricks-lab-ops/scripts/`: operational smoke-test scripts
- `cdc_gold/`: dbt project for Gold-layer models and tests
- `docker-compose.yml`: local Postgres/Kafka/Debezium stack

## Local Agents And Skills
This repo includes a local agent/skill catalog. Use it as part of normal work.

### How To Use Them
- Check `skills/` before non-trivial Databricks, CDC, dbt, notebook, or ops work.
- For role-based skills, read the matching file in `Agents/` before substantive implementation.
- If multiple roles apply, state the sequence you are using and keep responsibilities separated.
- Prefer the local repo-specific workflow skill over ad hoc commands when operating the stack.

### Most Relevant Local Agents
- `Agents/engineering-data-engineer.md`: best default for CDC, dbt, pipeline, and data reliability work
- `Agents/databricks_architect.md`: architecture choices for Databricks and lakehouse topology
- `Agents/lakehouse_data_architect.md`: Bronze/Silver/Gold contracts and model boundaries
- `Agents/databricks-notebook-publisher.md`: publish local notebooks into Databricks workspaces
- `Agents/databricks-job-operator.md`: run jobs, capture `run_id`, and track terminal state
- `Agents/databricks-notebook-remediator.md`: diagnose failed notebook runs and fix forward
- `Agents/databricks-data-quality-analyst.md`: validate output quality beyond job success
- `Agents/databricks-dq-automation.md`: run stored SQL DQ checks after pipeline changes
- `Agents/spark_performance_engineer.md`: Spark tuning and execution-performance review
- `Agents/evidenceqa.md` and `Agents/testing-reality-checker.md`: evidence-based QA and final readiness checks
- `Agents/confluence-documentation-generator.md`: generate Confluence-ready docs with Mermaid diagrams

### Most Relevant Local Skills
- `skills/docker-databricks-lab-ops/`: primary repo-specific operational workflow; use for stack bring-up, connector registration, generators, ngrok/Kafka wiring, Databricks runs, and smoke tests
- `skills/engineering-data-engineer/`: activate the repo's data-engineer specialization
- `skills/lakehouse-data-architect/`: use for medallion design and CDC model decisions
- `skills/databricks-notebook-publisher/`: use before publishing notebooks to the workspace
- `skills/databricks-job-operator/`: use when executing and monitoring Databricks jobs
- `skills/databricks-notebook-remediator/`: use when Databricks runs fail
- `skills/databricks-data-quality-analyst/`: use when validating table or notebook outputs
- `skills/databricks-dq-automation/`: run repo-managed SQL checks in `dq_queries/silver/` after pipeline updates
- `skills/testing-reality-checker/`: use for final confidence checks before calling work done
- `skills/confluence-documentation-generator/`: use to generate Confluence-ready docs with Mermaid diagrams

### Recommended Sequencing
- Delivery flow: `engineering-data-engineer` -> `databricks-notebook-publisher` -> `databricks-job-operator`
- Failure flow: `databricks-job-operator` -> `databricks-notebook-remediator` -> `databricks-job-operator`
- Validation flow: `databricks-data-quality-analyst` -> `testing-reality-checker`
- Automated DQ flow: `databricks-dq-automation` after successful Silver or workflow updates
- Documentation flow: `confluence-documentation-generator` to generate Confluence-ready docs
- End-to-end ops flow: consult `skills/docker-databricks-lab-ops/` first

### Notes
- `Agents/agents-orchestrator.md` describes a broader multi-agent pipeline and can be used when a task spans planning, implementation, remediation, and QA.
- Some agent files are generic imports from a wider catalog; prioritize the Databricks, lakehouse, data engineering, and QA roles for this repository.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alexeyban/databricks-lab](https://github.com/alexeyban/databricks-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
