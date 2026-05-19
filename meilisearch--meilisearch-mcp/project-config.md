---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 🚨 IMPORTANT: Development Workflow Guidelines

**ALL coding agents MUST follow these mandatory guidelines for every task.**

### 🔄 Fresh Start Protocol

**BEFORE starting ANY new task or issue:**

1. **Always start from latest main**:
   ```bash
   git checkout main
   git pull origin main  # Ensure you have the latest changes
   git checkout -b feature/your-branch-name  # Create new branch
   ```

2. **Verify clean state**:
   ```bash
   git status  # Should show clean working directory
   ```

3. **Never carry over unrelated changes** from previous work
4. **Each task gets its own focused branch** from latest main

### 🎯 Focused Development Rules

**ONLY make changes directly related to the specific task/issue:**

- ✅ **DO**: Add/modify code that solves the specific issue
- ✅ **DO**: Add focused tests for the specific functionality
- ✅ **DO**: Update documentation if specifically required
- ❌ **DON'T**: Include formatting changes to unrelated files
- ❌ **DON'T**: Add comprehensive test suites unless specifically requested
- ❌ **DON'T**: Refactor unrelated code
- ❌ **DON'T**: Include previous work from other branches

### 📋 Task Assessment Phase

Before writing any code, determine scope:

1. **Read the issue/task carefully** - understand exact requirements
2. **Identify minimal changes needed** - what files need modification?
3. **Plan focused tests** - only for the specific functionality being added
4. **Avoid scope creep** - resist urge to "improve" unrelated code

### 🧪 Test-Driven Development (TDD) Approach

When tests are required for the specific task:

```bash
# 1. Write failing tests FIRST (focused on the issue)
python -m pytest tests/test_specific_issue.py -v  # Should fail

# 2. Write minimal code to make tests pass
# Edit ONLY files needed for the specific issue

# 3. Run tests to verify they pass
python -m pytest tests/test_specific_issue.py -v  # Should pass

# 4. Refactor if needed, but stay focused
```

### 📝 Commit Standards

**Each commit should be atomic and focused:**

```bash
# Format only the files you changed
black src/specific_file.py tests/test_specific_file.py

# Run tests to ensure no regressions
python -m pytest tests/ -v

# Commit with descriptive message
git add src/specific_file.py tests/test_specific_file.py
git commit -m "Fix issue #X: Brief description of what was fixed"
```

### 🚫 What NOT to Include in PRs

- Formatting changes to files you didn't functionally modify
- Test files not related to your specific task
- Refactoring of unrelated code
- Documentation updates not specifically requested
- Code from previous branches or incomplete work

### ✅ PR Quality Checklist

Before creating PR, verify:
- [ ] Branch created from latest main
- [ ] Only files related to the specific issue are modified
- [ ] Tests pass and are focused on the issue
- [ ] Commit messages are clear and specific
- [ ] No unrelated formatting or code changes
- [ ] PR description clearly links to the issue being solved

**⚠️ PRs with unrelated changes will be rejected and must be redone.**

## Project Overview

This is a **Model Context Protocol (MCP) server** for Meilisearch, allowing LLM interfaces like Claude to interact with Meilisearch search engines. The project implements a Python-based MCP server that provides comprehensive tools for index management, document operations, search functionality, and system monitoring.

## Development Commands

### Environment Setup
```bash
# Create virtual environment and install dependencies
uv venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
uv pip install -e .

# Install development dependencies
uv pip install -r requirements-dev.txt
```

### Testing (MANDATORY for all development)
```bash
# Run all tests (required before any commit)
python -m pytest tests/ -v

# Run specific test file
python -m pytest tests/test_mcp_client.py -v

# Run tests with coverage (required for new features)
python -m pytest --cov=src tests/

# Watch mode for development (optional)
pytest-watch tests/
```

### Code Quality (MANDATORY before commit)
```bash
# Format code (required before commit)
black src/ tests/

# Check formatting without applying
black --check src/ tests/

# Run the MCP server locally for testing
python -m src.meilisearch_mcp

# Test with MCP Inspector
npx @modelcontextprotocol/inspector python -m src.meilisearch_mcp
```

### Prerequisites for Testing
- **Meilisearch server** must be running on `http://localhost:7700`
- **Docker option**: `docker run -d -p 7700:7700 getmeili/meilisearch:v1.28`
- **Node.js** for MCP Inspector testing

## Architecture

### Modular Manager Design
The codebase follows a modular architecture where functionality is organized into specialized managers:

```
MeilisearchClient
├── IndexManager      - Index creation, listing, deletion
├── DocumentManager   - Document CRUD operations
├── SettingsManager   - Index configuration management
├── TaskManager       - Asynchronous task monitoring
├── KeyManager        - API key management
└── MonitoringManager - Health checks and system metrics
```

### MCP Server Integration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [meilisearch/meilisearch-mcp](https://github.com/meilisearch/meilisearch-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
