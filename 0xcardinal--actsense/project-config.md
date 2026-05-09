---
trigger: always_on
description: This document provides comprehensive information for AI agents working with the actsense codebase.
---

# Agent Documentation for action-auditor

This document provides comprehensive information for AI agents working with the actsense codebase.

## Project Overview

**actsense** is a security auditing tool for GitHub Actions that:
- Analyzes GitHub Actions workflows and their dependencies
- Detects 65+ security vulnerabilities and best practice violations
- Visualizes action dependencies in an interactive graph
- Provides detailed security recommendations with links to comprehensive documentation

## Architecture

### Backend (Python/FastAPI)
- **FastAPI** web framework
- **Async/await** for concurrent operations
- **GitHub API** integration for fetching repository data
- **YAML parsing** for workflow files
- **Graph-based** dependency tracking
- **Facade pattern** for security checks (SecurityAuditor delegates to rules module)

### Frontend (React)
- **React 18** with functional components
- **ReactFlow** for graph visualization
- **Vite** for build tooling
- **Modern CSS** with component-scoped styles

### Documentation (Hugo)
- **Hugo static site generator** (Hextra theme)
- **Markdown-based** vulnerability documentation
- **Hosted at actsense.dev**

## Code Generation and Structure

### Code is Manually Written (Not Auto-Generated)

All code in this project is **manually written**, not auto-generated. The architecture follows these patterns:

1. **Rules Module Pattern**: Security checks are implemented in `backend/rules/security.py`
2. **Facade Pattern**: `backend/security_auditor.py` provides a facade that delegates to the rules module
3. **Separation of Concerns**: 
   - Rules contain the actual check logic
   - SecurityAuditor provides a clean API and orchestrates checks
   - Tests verify both layers independently

### Project Structure

```
backend/
├── main.py                 # FastAPI application entry point
├── security_auditor.py     # Facade for security checks (delegates to rules)
├── rules/
│   └── security.py         # Actual security check implementations (65+ checks)
├── github_client.py        # GitHub API client
├── workflow_parser.py      # YAML parsing and action extraction
├── graph_builder.py        # Dependency graph construction
├── repo_cloner.py          # Git repository cloning
├── analysis_storage.py     # JSON-based analysis persistence
├── config_loader.py        # Configuration loading (trusted publishers)
├── config.yaml             # Configuration file (trusted action publishers)
└── tests/                  # Comprehensive test suite
    ├── conftest.py         # Pytest fixtures
    ├── test_security_rules.py    # Tests for rules module
    ├── test_best_practices.py     # Tests for best practices
    ├── test_security_auditor.py   # Integration tests for facade
    └── ...

frontend/
└── src/
    ├── App.jsx                    # Main application
    └── components/
        ├── ActionGraph.jsx        # Graph visualization
        ├── NodeDetailsPanel.jsx  # Node details display
        ├── IssueDetailsModal.jsx  # Issue details modal
        └── ...

docs/
├── hugo.yaml                      # Hugo site configuration
├── content/
│   └── vulnerabilities/           # Markdown files for each vulnerability (67 files)
│       ├── unpinned_version.md
│       ├── potential_hardcoded_secret.md
│       └── ...
└── public/                        # Generated static site (gitignored)
```

## Key Components

### Backend Modules

#### `main.py`
- FastAPI application entry point
- API endpoints: `/api/audit`, `/api/analyses`, `/api/health`
- Repository and action auditing orchestration
- Dependency resolution logic
- Serves frontend static files in production

#### `github_client.py`
- GitHub API client with authentication
- Methods:
  - `get_repo_contents()` - Get repository files
  - `get_file_content()` - Get file content
  - `get_workflows()` - Get workflow files
  - `get_action_metadata()` - Get action.yml files
  - `get_latest_tag()` - Get latest version tag
  - `get_commit_date()` - Get commit date for SHA
  - `get_latest_tag_commit_date()` - Get latest tag's commit date
  - `parse_action_reference()` - Parse action references

#### `security_auditor.py` (Facade)
- **Facade pattern** that delegates to `rules/security.py`
- Provides clean API for security checks
- Main methods:
  - `audit_workflow()` - Audit a workflow file (async, orchestrates all checks)
  - `audit_action()` - Audit a single action
  - All `check_*()` methods delegate to `rules.security` module
- **Note**: This file contains mostly delegation code. Actual check logic is in `rules/security.py`

#### `rules/security.py` (Implementation)
- **Contains all actual security check implementations**
- 65+ security check functions
- Each check function:
  - Takes workflow/action data as input
  - Returns list of issue dictionaries
  - Each issue includes: type, severity, message, evidence, recommendation
  - Evidence includes link to actsense.dev documentation
- Functions are organized by category (secrets, permissions, injection, etc.)

#### `workflow_parser.py`
- YAML parsing and extraction
- Action reference detection
- Dependency extraction from composite actions
- Parses both workflow files and action.yml files

#### `graph_builder.py`
- Dependency graph construction

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0xCardinal/actsense](https://github.com/0xCardinal/actsense) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
