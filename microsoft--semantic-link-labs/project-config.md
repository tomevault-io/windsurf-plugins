---
trigger: always_on
description: This guide provides comprehensive instructions for AI agents working on the Semantic Link Labs (sempy_labs) codebase — an open-source Python library that extends the capabilities of [Semantic Link](https://learn.microsoft.com/en-us/fabric/data-science/semantic-link-overview) for Microsoft Fabric notebooks.
---

# Semantic Link Labs Development Agent Guide

This guide provides comprehensive instructions for AI agents working on the Semantic Link Labs (sempy_labs) codebase — an open-source Python library that extends the capabilities of [Semantic Link](https://learn.microsoft.com/en-us/fabric/data-science/semantic-link-overview) for Microsoft Fabric notebooks.

## Project Overview

Semantic Link Labs is a community-driven extension library designed for use in [Microsoft Fabric notebooks](https://learn.microsoft.com/fabric/data-engineering/how-to-use-notebook). It provides additional functionalities to work with:

- **Semantic models (datasets)** — Management, analysis, and migration tools
- **Workspaces** — Fabric and Power BI workspace operations
- **Lakehouses** — Lakehouse table management and shortcuts
- **Reports** — Report analysis, rebinding, and BPA
- **Capacities** — Capacity management and migration
- **And many more** — Notebooks, gateways, connections, etc.

### Key Features

| Feature Category | Description |
|-----------------|-------------|
| **TOM Wrapper** | Tabular Object Model wrapper for semantic model management |
| **Best Practice Analyzer** | Model and Report BPA for quality checks |
| **Direct Lake** | Migration, schema sync, and guardrails for Direct Lake models |
| **Vertipaq Analyzer** | Semantic model memory and performance analysis |
| **Admin APIs** | Admin-level operations for tenant management |
| **Migration Tools** | Capacity and model migration utilities |

### Dependency on Semantic Link

Semantic Link Labs depends on the `semantic-link-sempy` package as specified in `pyproject.toml`:

```toml
dependencies = [
    "semantic-link-sempy>=0.14.1",
    ...
]
```

The library uses the public APIs from `sempy.fabric` for:
- `FabricRestClient` — HTTP client for REST API calls
- `create_tom_server` — TOM server connection
- Standard Fabric context functions

**Important**: Only use public APIs from `sempy` as documented at:
- [Semantic Link Python API Reference](https://learn.microsoft.com/en-us/python/api/semantic-link-sempy/sempy.fabric)

---

## Source Code Layout

Main source code is located under `src/sempy_labs/`:

| Path | Description |
|------|-------------|
| `__init__.py` | Main module exports |
| `_helper_functions.py` | Shared utility functions (2800+ lines) |
| `_authentication.py` | Service Principal authentication |
| `_utils.py` | Item type mappings and constants |
| `_icons.py` | Icons, emojis, and UI constants |

### Core Submodules

| Submodule | Purpose |
|-----------|---------|
| `tom/` | TOM (Tabular Object Model) wrapper for semantic models |
| `admin/` | Admin API functions |
| `report/` | Report operations (BPA, rebind, export) |
| `lakehouse/` | Lakehouse operations (shortcuts, tables, schemas) |
| `directlake/` | Direct Lake model management |
| `migration/` | Model and capacity migration tools |

### Feature Modules (top-level)

| File | Feature |
|------|---------|
| `_model_bpa.py` | Model Best Practice Analyzer |
| `_model_bpa_rules.py` | BPA rule definitions |
| `_vertipaq.py` | Vertipaq Analyzer |
| `_workspaces.py` | Workspace management |
| `_capacities.py` | Azure capacity operations |
| `_refresh_semantic_model.py` | Model refresh operations |
| `_translations.py` | Model metadata translations |
| `_delta_analyzer.py` | Delta table analysis |

### Other Key Directories

| Directory | Purpose |
|-----------|---------|
| `tests/` | Unit tests |
| `notebooks/` | Sample helper notebooks |
| `docs/` | Sphinx documentation source |
| `.github/` | Issue templates and CI workflows |

---

## Development Guidelines

### Core Principles

1. **USE SKILLS PROACTIVELY** — Before starting ANY task, check the Skills Reference section below and read the relevant SKILL.md files. Skills contain domain-specific knowledge, patterns, fixtures, and best practices developed by the team. Failing to use skills leads to suboptimal implementations that require refactoring.
2. **Break down complexity** — Decompose large tasks into smaller, manageable subtasks. Use the planning-with-files skill for complex work.
3. **Understand before coding** — Fully understand requirements, context, and constraints before writing code.
4. **Clarify, don't assume** — Ask for specifics rather than making assumptions about requirements.
5. **Backward compatibility** — ALWAYS maintain backward compatibility for existing functions.
6. **Minimal changes** — Do NOT change existing code unless strongly necessary.
7. **Follow existing patterns** — Match the coding style and patterns already in the codebase.
8. **Document everything** — Every public function must have complete docstrings.

---

## Python Coding Standards

### Naming and Type Annotations

- Use descriptive, meaningful names for functions, variables, and classes.
- Include type hints for all function parameters and return values.
- Use the `typing` module for complex annotations (e.g., `List[str]`, `Dict[str, int]`, `Optional[T]`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/semantic-link-labs](https://github.com/microsoft/semantic-link-labs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
