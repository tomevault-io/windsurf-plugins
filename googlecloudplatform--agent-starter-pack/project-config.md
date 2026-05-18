---
trigger: always_on
description: > **Scope**: This document is for AI coding agents contributing to the **Agent Starter Pack repository itself** (the template generator). For guidance on working with **generated projects**, see [llm.txt](./llm.txt).
---

# Agent Starter Pack - AI Coding Agent Guide

> **Scope**: This document is for AI coding agents contributing to the **Agent Starter Pack repository itself** (the template generator). For guidance on working with **generated projects**, see [llm.txt](./llm.txt).

This document provides essential guidance, architectural insights, and best practices for AI coding agents tasked with modifying the Google Cloud Agent Starter Pack. Adhering to these principles is critical for making safe, consistent, and effective changes.

---

## Core Principles for AI Agents

1.  **Preserve and Isolate:** Your primary objective is surgical precision. Modify *only* the code segments directly related to the user's request. Preserve all surrounding code, comments, and formatting. Do not rewrite entire files or functions to make a small change.
2.  **Follow Conventions:** This project relies heavily on established patterns. Before writing new code, analyze the surrounding files to understand and replicate existing conventions for naming, templating logic, and directory structure.
3.  **Template-First Mindset:** ASP is a template generator. The CLI should remain lean with good defaults. Most features belong in templates, not CLI code.
4.  **Search Comprehensively:** A single change often requires updates in multiple places. When modifying configuration, variables, or infrastructure, you **must** search across the entire repository, including:
    *   `agent_starter_pack/base_templates/` (core templates by language)
    *   `agent_starter_pack/deployment_targets/` (environment-specific overrides)
    *   `.github/` and `.cloudbuild/` (CI/CD workflows)
    *   `docs/` (user-facing documentation)

---

## Project Architecture Overview

### 4-Layer Template System

Template processing follows this hierarchy (later layers override earlier ones):

| Layer | Directory | Purpose |
|-------|-----------|---------|
| 1. Base | `agent_starter_pack/base_templates/<language>/` | Core Jinja scaffolding (Python, Go, more coming) |
| 2. Deployment | `agent_starter_pack/deployment_targets/` | Environment overrides (cloud_run, gke, agent_engine) |
| 3. Frontend | `agent_starter_pack/frontends/` | UI-specific files |
| 4. Agent | `agent_starter_pack/agents/*/` | Agent-specific logic and configurations |

**Rule**: Always place changes in the correct layer. Check if deployment targets need corresponding updates.

### Key Directory Structure

```
agent_starter_pack/
├── agents/                    # Agent-specific files
│   ├── adk/                   # Base ADK agent (Python)
│   ├── adk_a2a/               # A2A-enabled ADK agent
│   ├── adk_go/                # Base ADK agent (Go)
│   ├── adk_live/              # Real-time multimodal agent
│   ├── agentic_rag/           # RAG agent
│   └── langgraph/             # LangGraph-based agent
├── base_templates/            # Core Jinja templates by language
│   ├── python/                # Python project template
│   │   ├── {{cookiecutter.agent_directory}}/
│   │   ├── deployment/
│   │   ├── tests/
│   │   └── Makefile
│   └── go/                    # Go project template
├── deployment_targets/        # Environment-specific overrides
│   ├── agent_engine/          # Agent Engine deployment
│   ├── cloud_run/             # Cloud Run deployment
│   └── gke/                   # GKE Autopilot deployment
├── frontends/                 # UI templates
└── cli/                       # CLI implementation
    ├── commands/              # create, setup-cicd, enhance, etc.
    └── utils/                 # Template processing, helpers
```

### When to Modify What

| Change Type | Where to Modify | Also Check |
|-------------|-----------------|------------|
| Affects ALL generated projects | `base_templates/<language>/` | Deployment targets for conflicts |
| Deployment-specific logic | `deployment_targets/<target>/` | Base templates for shared code |
| Agent-specific feature | `agents/<agent>/` | Other agents for consistency |
| New CLI flag/command | `cli/commands/` | `cli/utils/` for shared logic |
| CI/CD changes | Both `.github/` AND `.cloudbuild/` | Keep in sync |
| Documentation | `docs/` | README.md for overview changes |

### Template Processing Flow

1.  **Variable resolution** from `cookiecutter.json`
2.  **File copying** (base → deployment → frontend → agent overlays)
3.  **Jinja2 rendering** of file content
4.  **File/directory name rendering** (Jinja in filenames)

### Cross-File Dependencies

Changes often require coordinated updates:
- **Configuration**: `templateconfig.yaml` → `cookiecutter.json` → rendered templates
- **CI/CD**: `.github/workflows/` ↔ `.cloudbuild/` (must stay in sync)
- **Infrastructure**: Base terraform → deployment target overrides

---

## Template Development Workflow

Template changes require a specific workflow because you're modifying Jinja templates, not regular source files.

> **Note:** This workflow applies to both Python and Go templates. Both use Jinja templating with the same patterns (`{{cookiecutter.*}}`, `{% if %}`, etc.).

### Step-by-Step Process

#### 1. Generate a Test Instance

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GoogleCloudPlatform/agent-starter-pack](https://github.com/GoogleCloudPlatform/agent-starter-pack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
