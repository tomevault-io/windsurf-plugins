---
trigger: always_on
description: Create best-in-class repository scaffolds with modern tooling, security, CI/CD, testing, and documentation. Works for code, data science, ontologies, APIs, and CLI projects.
---


# repo-scaffold-skill

## Overview

Use this skill to scaffold and maintain production-ready repositories that follow modern best practices. This skill guides you through creating repositories with:

- **Security-first foundation**: Dependency scanning, secrets detection, vulnerability management
- **Automated quality gates**: Testing, linting, type checking, coverage enforcement
- **Exceptional developer experience**: One-command setup, pre-commit hooks, IDE configuration
- **Comprehensive documentation**: API docs, tutorials, troubleshooting guides
- **Observable systems**: Structured logging, performance benchmarking, monitoring

Applicable to: Python libraries, APIs, CLI tools, data science projects, ontologies, LinkML schemas, and web applications.

---

# Quick Start

When user says: **"Set up new repo in this folder"**

Execute this workflow:

1. **Assess project type**: Ask what kind of project (library, API, CLI, data science, ontology)
2. **Initialize repository structure**: Create foundational files and directories
3. **Configure tooling**: Set up uv, ruff, mypy, pytest, pre-commit
4. **Set up CI/CD**: Create GitHub Actions workflows
5. **Create documentation**: README, CONTRIBUTING, SECURITY, docs structure
6. **Initialize git**: Create repository with initial commit
7. **Configure GitHub**: Set up branch protection, security features (if requested)

---

# Principles

## Favored Copier Templates

For specific project types, prefer these blessed templates:

* **LinkML schemas**: https://github.com/linkml/linkml-project-copier
* **General code projects**: https://github.com/monarch-initiative/monarch-project-copier
* **Ontologies**: https://github.com/INCATools/ontology-development-kit (ODK framework)
* **AI integrations**: https://github.com/ai4curation/github-ai-integrations

Use these templates as starting points. This skill complements them with additional best practices.

## Modern Tooling Stack

### Python Development
* **Package management**: `uv` (fast, modern dependency resolver)
* **Linting & formatting**: `ruff` (replaces flake8, black, isort)
* **Type checking**: `mypy` in strict mode
* **Testing**: `pytest` with `pytest-cov` for coverage
* **Pre-commit**: Automated quality checks before commit

### Build Systems
* **Preferred**: `just` (for modern, readable task definitions)
* **Alternative**: `make` (for pipelines and compatibility)

### Documentation
* **Preferred**: `mkdocs` with Material theme (simple, beautiful)
* **Alternative**: `sphinx` (for legacy projects)
* **Framework**: Diátaxis (tutorials, how-to, reference, explanation)

### CLI Frameworks
* **Preferred**: `typer` (modern, type-based, beautiful output)
* **Alternative**: `click` (mature, widely used)
* **Output**: `rich` (colorful, formatted terminal output)

---

# Repository Scaffolding Workflow

## Step 1: Project Assessment

Ask these questions to understand project requirements:

### Project Type
- Library (reusable package for others)
- Application (deployable service/tool)
- CLI tool (command-line interface)
- API service (REST/GraphQL API)
- Data science project (analysis, ML models)
- Ontology (OWL/RDF knowledge representation)
- LinkML schema (data modeling)
- Documentation site only

### Technical Details
- Programming language (Python version if applicable)
- Target audience (developers, researchers, end-users)
- Deployment target (PyPI, Docker, cloud service, none)
- External dependencies (databases, APIs, services)
- Special requirements (performance, security, compliance)

### Team Context
- Solo developer or team
- Contribution model (open source, internal, private)
- Existing conventions or templates to follow

## Step 2: Initialize Structure

### Basic Directory Structure

**For Python Library:**
```
project-name/
├── .github/
│   ├── workflows/
│   │   ├── ci.yml
│   │   ├── release.yml
│   │   └── security.yml
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.md
│   │   └── feature_request.md
│   └── pull_request_template.md
├── docs/
│   ├── index.md
│   ├── tutorials/
│   ├── how-to/
│   ├── reference/
│   └── explanation/
├── src/
│   └── project_name/
│       ├── __init__.py
│       ├── cli.py (if CLI tool)
│       └── core.py
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── fixtures/
│   └── conftest.py
├── .gitignore
├── .pre-commit-config.yaml
├── CHANGELOG.md
├── CODE_OF_CONDUCT.md
├── CONTRIBUTING.md
├── justfile (or Makefile)
├── LICENSE
├── mkdocs.yml
├── pyproject.toml
├── README.md
├── SECURITY.md
└── uv.lock
```

**For Data Science Project:**
```
project-name/
├── .github/workflows/
├── data/              # Git-ignored
│   ├── raw/
│   ├── processed/
│   └── results/
├── notebooks/         # Exploratory analysis
├── src/
│   └── project_name/
│       ├── data/
│       ├── features/
│       ├── models/
│       └── visualization/
├── tests/
├── .dvc/             # If using DVC
├── pyproject.toml
└── README.md
```

**For API Service:**
```
project-name/
├── .github/workflows/
├── src/
│   └── project_name/
│       ├── api/
│       │   ├── routes/
│       │   └── dependencies.py
│       ├── core/
│       │   ├── config.py
│       │   └── security.py

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shiong-tan/repo-scaffold-skill](https://github.com/shiong-tan/repo-scaffold-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
