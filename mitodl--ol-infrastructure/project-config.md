---
trigger: always_on
description: This is a **large-scale infrastructure-as-code monorepo** (467 Python files, ~91K lines of code) for managing MIT Open Learning's cloud infrastructure using Pulumi, Packer/PyInfra, and Concourse.
---

# Agent Instructions for ol-infrastructure Repository

This is a **large-scale infrastructure-as-code monorepo** (467 Python files, ~91K lines of code) for managing MIT Open Learning's cloud infrastructure using Pulumi, Packer/PyInfra, and Concourse.

Scoped `AGENTS.md` files exist inside `src/bilder/`, `src/ol_concourse/`, and `src/ol_infrastructure/` — read those when working inside those packages. This root file covers repo-wide concerns only.

---

## Quick Start

```bash
# Environment setup (one-time)
uv sync
pulumi login s3://mitol-pulumi-state

# Validate your changes
uv run ruff format src/        # Auto-format code
uv run ruff check src/         # Check code quality
uv run mypy src/               # Type check (75s runtime)
uv run pytest tests/           # Run tests
```

---

## Documentation Index

The AGENTS.md instructions have been organized into focused documents. Read the relevant sections for your task:

### 📋 [Repository Overview](docs/context/01-repository-overview.md)

- Repository structure and organization
- Directory layout and purpose
- Environment requirements
- Critical build constraints

**Use when:** Understanding the codebase structure or finding where code should live

### 🔨 [Build and Validation](docs/context/02-build-and-validation.md)

- Environment setup and dependency installation
- Linting, formatting, and type checking commands
- Pre-commit hooks and validation checklist
- Common build issues and solutions

**Use when:** Setting up environment, validating code, or fixing build errors

### 🔄 Pulumi Workflows → [`src/ol_infrastructure/AGENTS.md`](src/ol_infrastructure/AGENTS.md)

Directory roles, stack naming, component rules, key helpers, validation.
Deep reference: [docs/context/03-pulumi-workflows.md](docs/context/03-pulumi-workflows.md)

### 📦 Packer AMI Building → [`src/bilder/AGENTS.md`](src/bilder/AGENTS.md)

Build chain, PyInfra conventions, component reuse, HCL formatting.
Deep reference: [docs/context/04-packer-bilder.md](docs/context/04-packer-bilder.md)

### 🔄 Concourse Pipelines → [`src/ol_concourse/AGENTS.md`](src/ol_concourse/AGENTS.md)

Pipeline DSL, factory functions, Identifier rules, meta pipeline registration.

### 🔐 [Secrets Management](docs/context/05-secrets-management.md)

- SOPS encryption and KMS integration
- Working with encrypted secrets in git
- Pulumi secret integration
- Vault runtime secret access
- Secret file organization

**Use when:** Managing secrets, encrypting sensitive data, configuring SOPS

### 🎨 [Code Style & Conventions](docs/context/06-code-style-conventions.md)

- Python type hints and Pydantic models
- Import organization and formatting
- Pulumi conventions (projects, stacks, components)
- Packer HCL and PyInfra style
- Configuration management patterns

**Use when:** Writing code, reviewing pull requests, establishing patterns

### 🧪 [Testing Strategy](docs/context/07-testing-strategy.md)

- When to write unit, integration, and policy tests
- Unit testing with Pulumi mocks
- Integration testing with Automation API
- Test organization and best practices
- Testing checklist for AI agents

**Use when:** Creating tests, validating infrastructure code, testing components

### 🏗️ [Architecture Decisions (ADR)](docs/context/08-architecture-decisions.md)

- When to create Architecture Decision Records
- ADR creation process and template
- Best practices for documenting decisions
- Integration with development workflow

**Use when:** Making significant architectural changes, evaluating options

### 🐛 [Troubleshooting](docs/context/09-troubleshooting.md)

- Common build, linting, and type-checking issues
- Pulumi, Packer, and secrets management problems
- Testing and timeout issues
- Debugging tips and getting help

**Use when:** Encountering errors, investigating problems, debugging

---

## File Modification Quick Reference

| Task | Files to Modify | Guide |
|------|----------------|-------|
| Add AWS resource | `src/ol_infrastructure/infrastructure/aws/<service>/__main__.py` | [`src/ol_infrastructure/AGENTS.md`](src/ol_infrastructure/AGENTS.md) |
| Create new app | `src/ol_infrastructure/applications/<app>/` | [`src/ol_infrastructure/AGENTS.md`](src/ol_infrastructure/AGENTS.md) |
| Add reusable component | `src/ol_infrastructure/components/<category>/` | [`src/ol_infrastructure/AGENTS.md`](src/ol_infrastructure/AGENTS.md) + [Testing](docs/context/07-testing-strategy.md) |
| Modify AMI build | `src/bilder/images/<image>/deploy.py` | [`src/bilder/AGENTS.md`](src/bilder/AGENTS.md) |
| Add CI/CD pipeline | `src/ol_concourse/pipelines/<category>/` | [`src/ol_concourse/AGENTS.md`](src/ol_concourse/AGENTS.md) |
| Manage secrets | `src/bridge/secrets/<context>/` | [Secrets Management](docs/context/05-secrets-management.md) |
| Document architecture | `docs/adr/NNNN-title.md` | [Architecture Decisions](docs/context/08-architecture-decisions.md) |
| Write tests | `tests/unit/` or `tests/integration/` | [Testing Strategy](docs/context/07-testing-strategy.md) |

---

## Validation Checklist

Before submitting changes:

```bash
uv sync
uv run ruff format src/
uv run ruff check src/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mitodl/ol-infrastructure](https://github.com/mitodl/ol-infrastructure) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
