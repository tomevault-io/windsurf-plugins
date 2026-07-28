---
trigger: always_on
description: > **Purpose**: This file guides AI assistants and developers working on HealthChain. It encodes coding standards, constraints, and workflows to keep architecture and domain judgment in human hands. It's a working document that will be updated as the project evolves.
---

# HealthChain - Claude Code Context

> **Purpose**: This file guides AI assistants and developers working on HealthChain. It encodes coding standards, constraints, and workflows to keep architecture and domain judgment in human hands. It's a working document that will be updated as the project evolves.

## 0. Project Overview

HealthChain is an open-source Python framework for productionizing healthcare AI applications with native protocol understanding. It provides built-in FHIR support, real-time EHR connectivity, and deployment tooling for healthcare AI/ML systems.

**Key Problem**: EHR data is specific, complex, and fragmented. HealthChain eliminates months of custom integration work by understanding healthcare protocols and data formats out of the box.

**Target Users** (two audiences, one platform):
- Clinical AI builders and healthtech teams shipping AI into real healthcare systems — EHR integration, FHIR, reliability
- AI and agent engineers building on healthcare data — agents, LLMs, and models that read and write validated FHIR

For more background, see @README.md and @docs/index.md.

---

## 1. Non-Negotiable Golden Rules

| # | AI *may* do | AI *must NOT* do |
|---|-------------|------------------|
| G-0 | When unsure about implementation details or requirements, ask developer for clarification before making changes. | ❌ Write changes or use tools when you are not sure about something project specific, or if you don't have context for a particular feature/decision. |
| G-1 | Generate code inside `healthchain/` or explicitly pointed files. | ❌ Modify or create test files without explicit approval. |
| G-2 | For changes >200 LOC or >3 files, propose a plan and wait for confirmation. | ❌ Refactor large modules without human guidance. |
| G-3 | Follow lint/style configs (`pyproject.toml`, `.ruff.toml`). Use `ruff` for formatting. | ❌ Reformat code to any other style. |
| G-4 | Stay within the current task context. Inform the dev if it'd be better to start afresh. | ❌ Continue work from a prior prompt after "new task" – start a fresh session. |

---

## 2. Testing Discipline

| What | AI CAN Do | AI MUST NOT Do |
|------|-----------|----------------|
| Implementation | Generate business logic | Write new tests without confirmation |
| Test Planning | Suggest test scenarios and coverage gaps | Implement test code during design phase |
| Debugging | Analyze test failures and suggest fixes | Modify test expectations without approval |

**Key principle**: Tests encode business requirements and human intent. AI assistance is welcome for suggestions, maintenance, and execution, but new test creation always requires explicit confirmation.

---

## 3. Build, Test & Utility Commands

Use `uv` for all development tasks:

```bash
# Testing
uv run pytest

# Linting & Formatting
uv run ruff check . --fix              # Lint and auto-fix
uv run ruff format .                   # Format code

# Dependency Management
uv sync                                # Install/sync dependencies
uv add <package>                       # Add dependency
uv add --dev <package>                 # Add dev dependency

# Security
uv run pip-audit                       # Audit installed deps for known vulnerabilities
```

> Vulnerability fixes arrive via Dependabot security PRs; CI runs `pip-audit` advisory-only on PRs. Run `pip-audit` locally for a full on-demand check.

---

## 4. Coding Standards

- **Python**: 3.10-3.13, prefer sync for legacy EHR compatibility; async available for modern systems but use only when explicitly needed
- **Dependencies**: Pydantic v2 (<2.11.0), NumPy <2.0.0 (spaCy compatibility)
- **Environment**: Use `uv` to manage dependencies and run commands (`uv run <command>`)
- **Formatting**: `ruff` enforces project style
- **Typing**: Always use explicit type hints, even for obvious types; Pydantic v2 models for external data
- **Naming**:
  - Code: `snake_case` (functions/vars), `PascalCase` (classes), `SCREAMING_SNAKE` (constants)
  - Files: No underscores, e.g., `fhiradapter.py` not `fhir_adapter.py`
- **Error Handling**: Prefer specific exceptions over generic
- **Documentation**: Docstrings for public APIs only
- **Healthcare Standards**: Follow HL7 FHIR and CDS Hooks specifications
- **Testing**: Separate test files matching source file patterns. Use flat functions instead of classes for tests.

---

## 5. Project Layout & Core Components

```
healthchain/
├── cli.py        # CLI entrypoint
├── config/       # Configuration management
├── configs/      # YAML + Liquid configs/templates
├── fhir/         # FHIR utilities and helpers
├── gateway/      # API gateways (FHIR, CDS Hooks)
├── interop/      # Format conversion (FHIR ↔ CDA, etc.)
├── io/           # Data containers, adapters, mappers (external formats ↔ HealthChain)
├── models/       # Pydantic data models
├── pipeline/     # Pipeline components and NLP integrations
├── sandbox/      # CDS Hooks testing scenarios & data loaders
├── templates/    # Code generation templates
└── utils/        # Shared utilities


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [healthchainai/HealthChain](https://github.com/healthchainai/HealthChain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
