---
trigger: always_on
description: This document provides essential guidance, architectural insights, and best practices for AI coding agents tasked with modifying the Google Cloud Agent Starter Pack. Adhering to these principles is critical for making safe, consistent, and effective changes.
---

# Agent Starter Pack - Coding Agent Guide

This document provides essential guidance, architectural insights, and best practices for AI coding agents tasked with modifying the Google Cloud Agent Starter Pack. Adhering to these principles is critical for making safe, consistent, and effective changes.

## Core Principles for AI Agents

1.  **Preserve and Isolate:** Your primary objective is surgical precision. Modify *only* the code segments directly related to the user's request. Preserve all surrounding code, comments, and formatting. Do not rewrite entire files or functions to make a small change.
2.  **Follow Conventions:** This project relies heavily on established patterns. Before writing new code, analyze the surrounding files to understand and replicate existing conventions for naming, templating logic, and directory structure.
3.  **Search Comprehensively:** A single change often requires updates in multiple places. When modifying configuration, variables, or infrastructure, you **must** search across the entire repository, including:
    *   `src/base_template/` (the core template)
    *   `src/deployment_targets/` (environment-specific overrides)
    *   `.github/` and `.cloudbuild/` (CI/CD workflows)
    *   `docs/` (user-facing documentation)

## Project Architecture Overview

### Templating Engine: Cookiecutter + Jinja2

The starter pack uses **Cookiecutter** to generate project scaffolding from templates that are heavily customized with the **Jinja2** templating language. Understanding the rendering process is key to avoiding errors.

**Multi-Phase Template Processing:**

Templates are processed in a specific order. A failure at any stage will break the project generation.

1.  **Cookiecutter Variable Substitution:** Simple replacement of `{{cookiecutter.variable_name}}` placeholders with values from `cookiecutter.json`.
2.  **Jinja2 Logic Execution:** Conditional blocks (`{% if %}`), loops (`{% for %}`), and other logic are executed. This is the most complex and error-prone stage.
3.  **File/Directory Name Templating:** File and directory names containing Jinja2 blocks are rendered. For example, `{% if cookiecutter.cicd_runner == 'github_actions' %}.github{% else %}unused_github{% endif %}`.

### Key Directory Structures

-   `src/base_template/`: This is the **core template**. Most changes that should apply to all generated projects should start here.
-   `src/deployment_targets/`: Contains files that **override or are added to** the `base_template` for a specific deployment target (e.g., `cloud_run`, `gke`, `agent_engine`). If a file exists in both `base_template` and a deployment target, the latter is typically used.
-   `agents/`: Contains pre-packaged, self-contained agent examples. Each has its own `.template/templateconfig.yaml` to define its specific variables and dependencies.
-   `src/cli/commands`: Contains the logic for the CLI commands, such as `create` and `setup-cicd`.

### CLI Commands

-   `create.py`: Handles the creation of new agent projects. It orchestrates the template processing, configuration merging, and deployment target selection.
-   `setup_cicd.py`: Automates the setup of the CI/CD pipeline. It interacts with `gcloud` and `gh` to configure GitHub repositories, Cloud Build triggers, and Terraform backend.

### Template Processing

-   `template.py`: Located in `src/cli/utils`, this script contains the core logic for processing the templates. It copies the base template, overlays the deployment target files, and then applies the agent-specific files.

## Critical Jinja Templating Rules

Failure to follow these rules is the most common source of project generation errors.

### 1. Block Balancing
**Every opening Jinja block must have a corresponding closing block.** This is the most critical rule.

-   `{% if ... %}` requires `{% endif %}`
-   `{% for ... %}` requires `{% endfor %}`
-   `{% raw %}` requires `{% endraw %}`

**Correct:**
```jinja
{% if cookiecutter.deployment_target == 'cloud_run' %}
  # Cloud Run specific content
{% endif %}
```

### 2. Variable Usage
Distinguish between substitution and logic:

-   **Substitution (in file content):** Use double curly braces: `{{ cookiecutter.project_name }}`
-   **Logic (in `if`/`for` blocks):** Use the variable directly: `{% if cookiecutter.use_alloydb %}`

### 3. Whitespace Control
Jinja is sensitive to whitespace. Use hyphens to control newlines and prevent unwanted blank lines in rendered files.

-   `{%-` removes whitespace before the block.
-   `-%}` removes whitespace after the block.

**Example:**
```jinja
{%- if cookiecutter.some_option %}
option = true
{%- endif %}
```

## Terraform Best Practices

### Unified Service Account (`app_sa`)
The project uses a single, unified application service account (`app_sa`) across all deployment targets to simplify IAM management.

-   **Do not** create target-specific service accounts (e.g., `cloud_run_sa`).
-   Define roles for this account in `app_sa_roles`.
-   Reference this account consistently in all Terraform and CI/CD files.

### Resource Referencing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mihai-pompiliu/agent-starter-pack](https://github.com/mihai-pompiliu/agent-starter-pack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-10 -->
