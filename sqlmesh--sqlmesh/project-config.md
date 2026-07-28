---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Agent-Based Development Workflow

Every time the user requests a feature or bug fix, you MUST follow the process below:

### Development Process

1. **Understanding The Task**: Use the `developer` agent to understand what the user is asking for and to read GitHub issues
2. **Feature Development & Bug Fixes**: Use the `developer` agent for implementing features and fixing bugs. IMPORTANT: Always begin by writing a failing test (or tests) that reflects the expected behavior
3. **Code Review**: After development work, invoke the `code-reviewer` agent to review the implementation
4. **Iteration**: Use the `developer` agent again to address feedback from the code reviewer
5. **Repeat**: Continue the developer → code-reviewer cycle until no more feedback remains
6. **Documentation**: If the feature or bug fix requires documentation updates, invoke the `technical-writer` agent

IMPORTANT: Make sure to share the project overview, architecture overview, and other concepts outlined below with the agent when it is invoked.

### Agent Responsibilities

**Developer Agent**:
- Understands a feature request or a reported issue
- Implements new features following SQLMesh's architecture patterns
- Fixes bugs with proper understanding of the codebase
- Writes comprehensive tests following SQLMesh's testing conventions
- Follows established code style and conventions

**Code-Reviewer Agent**:
- Reviews implementation for quality and architectural compliance
- Identifies potential issues, edge cases, and improvements
- Ensures adherence to SQLMesh patterns and best practices
- Validates test coverage and quality

**Technical-Writer Agent**:
- Creates and updates user-facing documentation
- Writes API documentation for new features
- Updates existing docs after code changes
- Creates migration guides and deep-dive technical explanations

## Project Overview

SQLMesh is a next-generation data transformation framework that enables:
- Virtual data environments for isolated development without warehouse costs
- Plan/apply workflow (like Terraform) for safe deployments
- Multi-dialect SQL support with automatic transpilation
- Incremental processing to run only necessary transformations
- Built-in testing and CI/CD integration

**Requirements**: Python >= 3.9 (Note: Python 3.13+ is not yet supported)

## Essential Commands

### Environment setup
```bash
# Create and activate a Python virtual environment (Python >= 3.9, < 3.13)
python -m venv .venv
source ./.venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install development dependencies
make install-dev

# Setup pre-commit hooks (important for code quality)
make install-pre-commit
```

### Common Development Tasks
```bash
# Run linters and formatters (ALWAYS run before committing)
make style

# Fast tests for quick feedback during development
make fast-test

# Slow tests for comprehensive coverage
make slow-test

# Run specific test file
pytest tests/core/test_context.py -v

# Run tests with specific marker
pytest -m "not slow and not docker" -v

# Build package
make package

# Serve documentation locally
make docs-serve
```

### Engine-Specific Testing
```bash
# DuckDB (default, no setup required)
make duckdb-test

# Other engines require credentials/Docker
make snowflake-test  # Needs SNOWFLAKE_* env vars
make bigquery-test   # Needs GOOGLE_APPLICATION_CREDENTIALS
make databricks-test # Needs DATABRICKS_* env vars
```

### UI Development
```bash
# In web/client directory
pnpm run dev   # Start development server
pnpm run build # Production build
pnpm run test  # Run tests

# Docker-based UI
make ui-up     # Start UI in Docker
make ui-down   # Stop UI
```

## Architecture Overview

### Core Components

**sqlmesh/core/context.py**: The main Context class orchestrates all SQLMesh operations. This is the entry point for understanding how models are loaded, plans are created, and executions happen.

**sqlmesh/core/model/**: Model definitions and kinds (FULL, INCREMENTAL_BY_TIME_RANGE, SCD_TYPE_2, etc.). Each model kind has specific behaviors for how data is processed.

**sqlmesh/core/snapshot/**: The versioning system. Snapshots are immutable versions of models identified by fingerprints. Understanding snapshots is crucial for how SQLMesh tracks changes.

**sqlmesh/core/plan/**: Plan building and evaluation logic. Plans determine what changes need to be applied and in what order.

**sqlmesh/core/engine_adapter/**: Database engine adapters provide a unified interface across 16+ SQL engines. Each adapter handles engine-specific SQL generation and execution.

### Key Concepts

1. **Virtual Environments**: Lightweight branches that share unchanged data between environments, reducing storage costs and deployment time.

2. **Fingerprinting**: Models are versioned using content-based fingerprints. Any change to a model's logic creates a new version.

3. **State Sync**: Manages metadata across different backends (can be stored in the data warehouse or external databases).

4. **Intervals**: Time-based partitioning system for incremental models, tracking what data has been processed.

## Important Files


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SQLMesh/sqlmesh](https://github.com/SQLMesh/sqlmesh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
