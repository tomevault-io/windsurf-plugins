---
trigger: always_on
description: The **Solution Application Framework (SAF)** is a Python-based toolkit for creating production-ready simulation web applications. It gives method developers and engineers the building blocks to turn complex, multi-domain simulation workflows into guided, shareable web apps — without needing to become full-stack software engineers.
---

# Overview

The **Solution Application Framework (SAF)** is a Python-based toolkit for creating production-ready simulation web applications. It gives method developers and engineers the building blocks to turn complex, multi-domain simulation workflows into guided, shareable web apps — without needing to become full-stack software engineers.

At its core, SAF provides the **Guided Low-Code Workflow (GLOW)** engine — a Python framework that abstracts complex backend operations, orchestrates Ansys product integrations, and exposes a unified API for building, running, and managing solution applications.

## Key capabilities

- **Templated, production-ready apps** with batteries-included project structure, dependency management, and observability
- **Desktop-ready** with code encryption, standalone executable installers, and fully offline installation
- **Native Ansys ecosystem integration** with PyAnsys SDKs, HPC Platform Services (HPS), Minerva, and the Data Repository STC
- **Open architecture** — use any frontend framework (Dash, React, Angular) and connect to third-party software via REST APIs
- **Flexible deployment** — write once, deploy anywhere (desktop, on-premises, or cloud) without changing source code

## Repository model

This repository is a **monorepo** containing 12 independent packages. Each package has its own `pyproject.toml`, `poetry.lock`, `.pre-commit-config.yaml`, and release lifecycle.

- **Python**: >=3.11, <4.0 (CI tests on 3.11–3.14)
- **Build system**: Poetry (per-package), uv (root-level only)
- **License**: Apache-2.0 (all new files require a license header with `start_year=2026`)

# Repository layout

```
saf/
├── .pre-commit-config.yaml          # Root-level hooks (gitleaks, zizmor, trailing-whitespace, yamlfmt, license headers)
├── pyproject.toml                    # Root project — doc dependencies only, no publishable code
├── doc/styles/Vocab/ANSYS/accept.txt # Shared spelling dictionary (Vale + codespell)
├── architecture/                     # Monorepo design docs (architecture.md, CI docs)
├── doc/                              # Root-level Sphinx documentation
├── .github/
│   ├── workflows/
│   │   ├── ci_cd_pr.yml              # Main CI: label-gated per-package style, build, test
│   │   ├── _test.yml                 # Reusable test workflow (matrix from JSON definitions)
│   │   ├── ci_cd_release.yml         # Release workflow (one package at a time)
│   │   ├── certification.yml         # Certification workflow (multi-Python)
│   │   └── tests_groups_definitions/ # Per-package JSON test session configs
│   ├── labeler.yml                   # Maps changed paths → PR labels
│   └── labels.yml                    # Canonical label definitions
└── packages/                         # All 12 Python packages
    ├── bdm-python-api/
    ├── bdm-python-shared-volume/
    ├── dash-super-components/
    ├── glow-engine/
    ├── saf-cli/
    ├── saf-desktop-installer/
    ├── saf-desktop-orchestrator/
    ├── saf-iam-oidc/
    ├── saf-product-configuration/
    ├── saf-product-manager/
    ├── saf-templates/
    └── saf-testing/
```

Each package follows a uniform internal structure: `src/`, `tests/`, `pyproject.toml`, `poetry.lock`, `.pre-commit-config.yaml`.

# Build, test, and validate

## Prerequisites

- **Long paths** must be enabled (`git config --global core.longpaths true`; also enable in Windows registry).
- **uv** for root-level tasks: `pip install uv`
- **Poetry 2.3.2** for package-level tasks.

## Root-level tasks (run from repo root, use uv)

```bash
# Pre-commit (root hooks: gitleaks, zizmor, trailing-whitespace, yamlfmt, license headers)
uv venv .venv --python 3.12
.venv\Scripts\activate        # Windows
uv pip install pre-commit==4.6.0
uv run pre-commit run --all-files

# Documentation
uv sync --group doc
uv run sphinx-build doc/source doc/build/html
```

## Package-level tasks (run from packages/<name>, use Poetry)

```bash
cd packages/<package-name>

# Install all dependencies
poetry install --all-extras

# Some packages need additional groups for style checks:
#   glow-engine:              poetry install --with tests,style --all-extras
#   saf-desktop-installer:    poetry install --with tests,style --all-extras
#   saf-desktop-orchestrator: poetry install --with tests,dev --all-extras
#   dash-super-components:    poetry install --with tests,style --all-extras

# Run tests
poetry run pytest
```

# Run package-level pre-commit (ruff, pyright, bandit, poetry-check, etc.)
Not all packages include `pre-commit` in their `pyproject.toml`. The dependency group varies by package:

| Group | Packages |
|-------|----------|
| `style` | glow-engine, saf-testing, dash-super-components |
| `dev` | saf-cli, saf-desktop-orchestrator, saf-product-manager, saf-product-configuration |
| *(not included)* | bdm-python-api, bdm-python-shared-volume, saf-iam-oidc, saf-desktop-installer, saf-templates |

For packages that include `pre-commit`:

```bash
cd packages/<component>


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ansys/saf](https://github.com/ansys/saf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
