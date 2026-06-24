---
trigger: always_on
description: Manages GitHub Actions WIF, service accounts, APIs, storage, and IAM for the CI/CD infrastructure using `agents-cli-test-*` projects.
---

# agents-cli — AI Coding Agent Guide

> **Scope**: This document is for AI coding agents contributing to the **Agents CLI in Agent Platform** repository itself (the CLI tool that scaffolds, develops, evaluates, and deploys ADK agents). For guidance on working with **generated projects**, see the bundled skills under `skills/`.

---

## Core Principles for AI Agents

1.  **Preserve and Isolate:** Modify *only* the code segments directly related to the user's request — preserve all surrounding code, comments, and formatting.
2.  **Follow Conventions:** Analyze existing patterns before writing new code; replicate naming, templating logic, and directory structure conventions.
3.  **Template-First Mindset:** The CLI should remain lean with good defaults. Most features belong in templates, not CLI code.
4.  **Search Comprehensively:** A single change often requires updates in multiple places across templates, CLI code, and CI/CD configuration.

---

## Project Architecture Overview

### Package Structure

```
src/google/agents/cli/
├── __init__.py              # __version__
├── main.py                  # Root Click group, registers all commands
├── _output.py               # emit(), ExitCode enum (JSON structured output)
├── _project.py              # read_project_config() — reads configuration manifest from agents-cli-manifest.yaml
├── _runner.py               # run() — unified subprocess helper
├── _trust.py                # require_confirmation() decorator
├── auth.py                  # Authentication helpers
│
├── setup/                   # Skills installation + auth commands
│   ├── cmd_setup.py         # agents-cli setup — install bundled skills via npx
│   ├── cmd_update.py        # agents-cli update — update skills via npx
│   └── cmd_auth.py          # agents-cli login/status
│
├── scaffold/                # Project scaffolding (vendored from Agent Starter Pack)
│   ├── commands/
│   │   ├── create.py        # agents-cli scaffold create (alias: agents-cli create)
│   │   ├── enhance.py       # agents-cli scaffold enhance
│   │   └── upgrade.py       # agents-cli scaffold upgrade
│   ├── utils/               # Template processing, helpers, lock generation
│   ├── agents/              # Agent templates (adk, adk_a2a, agentic_rag)
│   ├── base_templates/      # Base project templates (python/)
│   ├── deployment_targets/  # Deployment overrides (agent_runtime, cloud_run, gke)
│   └── resources/           # Locks, docs, IDX configs
│
├── dev/                     # Development commands (thin wrappers)
│   ├── cmd_playground.py    # agents-cli playground — start local playground
│   ├── cmd_lint.py          # agents-cli lint — ruff check + format
│   └── cmd_install.py       # agents-cli install — uv sync
│
├── run/                     # Run command
│   └── cmd_run.py           # agents-cli run — run agent with a single prompt
│
├── data/                    # Data/RAG commands
│   ├── _helpers.py             # Shared helpers (resolve_project_id, require_rag_project, …)
│   └── cmd_data_ingestion.py   # agents-cli data-ingestion
│
├── eval/                    # Evaluation commands
│   ├── cmd_analyze.py       # agents-cli eval analyze
│   ├── cmd_compare.py       # agents-cli eval compare
│   ├── cmd_dataset.py       # agents-cli eval dataset
│   ├── cmd_generate.py      # agents-cli eval generate
│   ├── cmd_grade.py         # agents-cli eval grade
│   ├── cmd_metric.py        # agents-cli eval metric
│   └── cmd_optimize.py      # agents-cli eval optimize
│
├── deploy/                  # Deployment commands
│   ├── cmd_deploy.py        # agents-cli deploy — dispatches by deployment_target
│   └── agent_runtime.py      # Agent Runtime deployment logic
│
├── infra/                   # Infrastructure commands
│   ├── cmd_infra.py         # agents-cli infra single-project — single-project terraform
│   ├── cmd_cicd.py          # agents-cli infra cicd — CI/CD pipeline + staging/prod infra
│   └── cmd_datastore.py     # agents-cli infra datastore — RAG datastore provisioning
│
├── publish/                 # Publish commands
│   └── cmd_publish.py       # agents-cli publish gemini-enterprise
│
├── info/                    # Info command
│   └── cmd_info.py          # agents-cli info — show project config and CLI version
│
└── skills/                  # SKILLS_DATA_DIR path resolution
    └── __init__.py          # Resolves to root skills/ (source) or bundled data/ (wheel)

skills/                      # Bundled IDE skills at repo root
├── google-agents-cli-workflow/
├── google-agents-cli-adk-code/
├── google-agents-cli-scaffold/
├── google-agents-cli-eval/
├── google-agents-cli-deploy/
├── google-agents-cli-publish/
└── google-agents-cli-observability/
```

### Bundled Skills

Skills are installed to coding agents via `agents-cli setup` and provide context-specific guidance for working with generated agent projects.

- **google-agents-cli-workflow** — Always-active development lifecycle guide. Covers the spec-driven workflow (understand, build, evaluate, deploy), code preservation rules, model selection, and troubleshooting.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agents-cli-dev-bot/agents-cli-shadow-ci-win-w73hxa55](https://github.com/agents-cli-dev-bot/agents-cli-shadow-ci-win-w73hxa55) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
