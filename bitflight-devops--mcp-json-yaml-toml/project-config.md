---
trigger: always_on
description: **mcp-json-yaml-toml** is a token-efficient, schema-aware MCP (Model Context Protocol) server for safely reading and modifying JSON, YAML, and TOML files. It provides AI assistants with structured data manipulation tools that preserve comments and formatting while enforcing schema validation.
---

# Copilot Instructions - mcp-json-yaml-toml

## Repository Overview

**mcp-json-yaml-toml** is a token-efficient, schema-aware MCP (Model Context Protocol) server for safely reading and modifying JSON, YAML, and TOML files. It provides AI assistants with structured data manipulation tools that preserve comments and formatting while enforcing schema validation.

- **Size**: ~2.4MB, 28 Python files
- **Stack**: Python 3.11-3.14, FastMCP, uv package manager, hatchling build backend
- **Architecture**: 100% local processing, no API keys required
- **Cross-Platform**: Works seamlessly on Windows, Linux, and macOS (any system where yq binary is supported)
- **Key Features**: yq-based transformations, SchemaStore.org integration, LMQL constraint validation

## Prerequisites & Setup

### Required Tools

- **Python**: 3.11-3.14 (CI tests against 3.11, 3.12, 3.13, 3.14)
- **uv**: Package manager (replaces pip/poetry/pipenv)
- **Node.js**: Required for markdown/prettier linting (npx commands)

### Initial Setup

**ALWAYS run this first after cloning:**

```bash
uv sync  # Installs all dependencies and dev-tools (~30-60 seconds)
```

This command:

- Creates `.venv/` virtual environment
- Installs all Python dependencies from `pyproject.toml`
- Installs dev dependencies (pytest, ruff, mypy, basedpyright, prek)
- Is idempotent - safe to run multiple times

**NEVER use `pip install` or bare `python` commands. Always use `uv run <command>`.**

## Validation Commands (CI Enforcement)

**Recommended Approach**: Use `prek` to run all validation checks in one command:

```bash
uv run prek run --files <file1> <file2>  # For specific files
uv run prek run --all-files               # For all files (use sparingly)
```

The `prek` tool automatically runs all the checks below in parallel. For debugging or CI reference, individual commands are listed:

### Individual Validation Commands

These are automatically run by `prek` but can be executed separately if needed:

### 1. Code Formatting

```bash
uv run ruff format --check
```

**If it fails**: Run `uv run ruff format` to auto-fix, then commit.

### 2. Code Linting

```bash
uv run ruff check
```

**If it fails**: Run `uv run ruff check --fix` to auto-fix. Some issues require manual fixes.

### 3. Type Checking - Mypy

```bash
uv run mypy packages/ --show-error-codes
```

**Critical**: NEVER suppress type errors with `# type: ignore` for structural issues. Fix the root cause.

### 4. Type Checking - Basedpyright

```bash
uv run basedpyright packages/
```

### 5. Markdown Linting

```bash
npx -y markdownlint-cli2
```

Uses `.markdownlint-cli2.jsonc` for configuration.

### 6. YAML/JSON/Markdown Formatting

```bash
npx -y prettier --check "**/*.{md,json,yaml,yml}" --ignore-path .gitignore
```

**If it fails**: Run `npx -y prettier --write "**/*.{md,json,yaml,yml}" --ignore-path .gitignore` to auto-fix.

### 7. Tests

```bash
uv run pytest
```

- **Runs**: 350+ tests with parallel execution (`-n auto`)
- **Coverage**: Minimum 60% required (current: ~79%)
- **Location**: `packages/mcp_json_yaml_toml/tests/`
- **Run specific tests**: `uv run pytest -k <pattern>`

## Quick Validation Workflow

**After editing files, use prek for comprehensive validation:**

```bash
uv run prek run --files <file1> <file2>
```

**Example:**

```bash
uv run prek run --files packages/mcp_json_yaml_toml/server.py
```

**CRITICAL: NEVER use `--all-files` during feature work** - it causes diff pollution and rewrites git history.

## Project Architecture

### Directory Structure

```
mcp-json-yaml-toml/
├── packages/mcp_json_yaml_toml/     # Main package (ALL code here)
│   ├── server.py                    # MCP server & tool implementations
│   ├── yq_wrapper.py                # Binary management & yq execution
│   ├── schemas.py                   # Schema discovery & validation
│   ├── lmql_constraints.py          # LMQL constraint validation
│   ├── yaml_optimizer.py            # YAML anchor optimization
│   ├── toml_utils.py                # TOML utilities
│   ├── config.py                    # Configuration management
│   ├── version.py                   # Dynamic version from git tags
│   └── tests/                       # All test files (test_*.py)
│       ├── conftest.py              # Shared pytest fixtures
│       └── test_*.py                # Individual test modules
├── .github/workflows/               # CI/CD pipelines
│   ├── test.yml                     # Main CI (format/lint/type/test)
│   └── auto-publish.yml             # PyPI auto-publishing
├── docs/                            # Documentation
├── scripts/                         # Utility scripts
├── pyproject.toml                   # All configuration (build/lint/type/test)
├── .pre-commit-config.yaml          # Pre-commit hooks (uses prek)
├── AGENTS.md                        # Master instructions for AI agents
└── README.md                        # User documentation
```

### Key Configuration Files


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bitflight-devops/mcp-json-yaml-toml](https://github.com/bitflight-devops/mcp-json-yaml-toml) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
