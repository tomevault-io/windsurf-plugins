---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Patchsmith** is an AI-powered CLI tool that automates security vulnerability detection and fixing using CodeQL for analysis and Claude for intelligent processing. The project is designed with a layered architecture to support future SaaS evolution.

**Current Status**: Phase 1 (Foundation) complete - 69 tests passing, 62% coverage, all domain models implemented.

## Development Workflow

### Spec-Driven Development

Patchsmith uses **spec-driven development** for major release iterations. This ensures thorough planning, clear requirements, and systematic implementation.

**Specification Structure:**

Each major version has a dedicated specification folder in `specs/` with incremental versioning:

```
specs/
├── 1_initial_version/     # v0.1.0 specs
│   ├── requirements.md    # Full requirements specification
│   ├── design.md          # Technical design and architecture
│   └── tasks.md           # Implementation tasks and checklist
├── 2_next_version/        # v0.2.0 specs (future)
│   ├── requirements.md
│   ├── design.md
│   └── tasks.md
└── 3_future_version/      # v0.3.0 specs (future)
    └── ...
```

**Development Process:**

1. **Planning Phase** - Create spec folder for next version:
   - `requirements.md` - Define features, user stories, acceptance criteria
   - `design.md` - Design architecture, data models, API contracts
   - `tasks.md` - Break down into actionable tasks with checkboxes

2. **Implementation Phase** - Work through tasks systematically:
   - Check off tasks as completed in `tasks.md`
   - Update specs if requirements change during implementation
   - Keep specs accurate as living documentation

3. **Release** - Version is complete when all tasks checked off

**Spec File Purposes:**

- **requirements.md** - What we're building and why (product requirements)
- **design.md** - How we're building it (technical architecture, patterns, decisions)
- **tasks.md** - Detailed implementation checklist (broken down by component/phase)

**Important:** For large features or architectural changes, always consult the relevant spec files in `specs/` before making changes. These documents contain the authoritative design decisions and requirements.

## Development Commands

### Setup
```bash
poetry install                    # Install all dependencies
```

### Testing
```bash
poetry run pytest                                    # Run all tests
poetry run pytest tests/unit/models/                # Run specific test directory
poetry run pytest tests/unit/models/test_finding.py # Run single test file
poetry run pytest -v                                # Verbose output
poetry run pytest -k "test_name"                    # Run tests matching name
poetry run pytest --cov-report=html                 # Generate HTML coverage report
```

### Code Quality
```bash
poetry run mypy src/patchsmith    # Type checking (must pass)
poetry run ruff check src/        # Linting
poetry run ruff check --fix src/  # Auto-fix linting issues
poetry run black src/ tests/      # Format code
```

### Running the CLI
```bash
poetry run patchsmith --help      # Show CLI help
poetry run patchsmith --version   # Show version
```

## Architecture

Patchsmith uses a **layered architecture** designed to be presentation-agnostic:

```
Presentation Layer (CLI)
    ↓
Service Layer (Business Logic) ← Core of the application
    ↓
Adapter Layer (External Integrations: CodeQL, Claude, Git)
    ↓
Infrastructure Layer (Config, Logging, I/O)
```

### Key Architectural Principles

1. **Service Layer Independence**: The service layer (`src/patchsmith/services/`) contains all business logic and MUST NOT depend on the presentation layer (`cli/` or `presentation/`). Services accept progress callbacks and return domain models.

2. **Progress Callbacks**: Services emit progress events via callbacks rather than directly outputting to console. This allows the same service to work with CLI progress bars, WebSocket updates, or API responses.

3. **Dependency Injection**: Services receive their dependencies (adapters) through constructor injection, making them testable in isolation.

4. **Repository Pattern**: Data access is abstracted through repositories (`src/patchsmith/repositories/`). Currently file-based, designed to swap to database for SaaS.

5. **Domain Models First**: All data structures are Pydantic models (`src/patchsmith/models/`) with validation and serialization built-in.

## Project Structure

```
src/patchsmith/
├── cli/              # CLI commands (thin wrappers around services)
├── services/         # Business logic layer (presentation-agnostic)
├── adapters/         # External integrations (CodeQL, Claude, Git)
│   ├── codeql/      # CodeQL CLI wrapper
│   ├── claude/      # Claude AI agents
│   └── git/         # Git operations
├── core/            # Config management, orchestration
├── models/          # Pydantic domain models
│   ├── config.py    # Configuration models
│   ├── finding.py   # Security finding models (Severity, CWE, Finding)
│   ├── analysis.py  # Analysis result models
│   ├── project.py   # Project info models

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eschnou/patchsmith](https://github.com/eschnou/patchsmith) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
