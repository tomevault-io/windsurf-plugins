---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

**IntentGraph** is a static code analysis CLI tool optimized for AI agent context windows. It analyzes Python, JavaScript, and TypeScript codebases (with basic Go support) and generates token-efficient structured output that fits within AI agent limitations.

**Version**: 0.4.0 (Live on PyPI)
**Python**: 3.12+
**License**: MIT
**Status**: Production-ready with RepoSnapshot v1 (deterministic snapshots)

## Core Purpose

IntentGraph solves the **context window problem** for AI coding agents by:
- Pre-analyzing Python, JavaScript, and TypeScript codebases into minimal, AI-optimized output (~10KB vs 340KB full analysis)
- Providing function-level dependency tracking and code metrics (complexity, maintainability)
- Offering intelligent clustering for massive repositories that exceed token limits
- **Creating deterministic snapshots (v0.4.0)** with stable UUIDs and runtime environment detection
- Exposing a production-ready CLI with an AI integration framework for future expansion

## What's Real vs. What's Framework

**✅ Fully Working:**
- Python, JavaScript, and TypeScript code analysis with complete AST parsing
- Dependency graph generation with cycle detection
- Code complexity and maintainability metrics
- Three-level output system (minimal, medium, full)
- Intelligent clustering for large codebases
- **RepoSnapshot v1 (v0.4.0)**: Deterministic snapshots with SHA256-based UUIDs, runtime detection, schema freeze
- Comprehensive CLI with rich options
- 147 pytest tests (114 passing, 29 new snapshot tests at 100%)

**🏗️ Framework/Scaffolding:**
- AI-native interface exists with structure in place
- Natural language query parsing works
- Semantic query execution uses basic file filtering (not deep AI understanding)
- Response optimization returns template data
- Task-aware optimization structure exists but needs full implementation

## Quick Commands

### Development Setup
```bash
# Clone and setup
git clone <repo-url>
cd intentgraph
python -m venv .venv
source .venv/bin/activate  # or .venv\Scripts\activate on Windows
pip install -e ".[dev]"
```

### Testing
```bash
# Run all tests with coverage
pytest --cov=intentgraph --cov-report=term-missing

# Run specific test file
pytest tests/test_domain/test_models.py -v

# Run tests excluding slow tests
pytest -m "not slow"

# Run only integration tests
pytest -m integration

# Run with specific coverage threshold
pytest --cov=intentgraph --cov-fail-under=90
```

### Code Quality
```bash
# Format code (ALWAYS run before committing)
ruff format .

# Lint and auto-fix
ruff check --fix .

# Type checking
mypy .

# Security scanning
bandit -r src/

# Run all quality checks
ruff format . && ruff check --fix . && mypy . && bandit -r src/
```

### CLI Usage (Primary Interface)
```bash
# Analyze current directory (minimal output, AI-friendly)
intentgraph .

# Generate cluster analysis for large codebases
intentgraph . --cluster --cluster-mode analysis

# Full analysis with all metadata
intentgraph . --level full --output analysis.json

# Multi-language analysis (Python, JavaScript, TypeScript)
intentgraph /path/to/repo --lang py,js,ts

# Check for circular dependencies
intentgraph . --show-cycles
```

### Programmatic Usage
```bash
# Traditional Python API (production-ready)
from intentgraph import RepositoryAnalyzer
analyzer = RepositoryAnalyzer(workers=4, include_tests=False)
result = analyzer.analyze(Path("/path/to/repo"))

# AI Integration Framework (experimental)
from intentgraph import connect_to_codebase
agent = connect_to_codebase("/path/to/repo", {"task": "bug_fixing"})
results = agent.query("Find high complexity files")
# Note: Query execution uses file-based filtering, not deep semantic analysis

# RepoSnapshot v1 (production-ready, v0.4.0+)
from intentgraph.snapshot import RepoSnapshotBuilder
builder = RepoSnapshotBuilder(Path("/path/to/repo"))
snapshot = builder.build()
json_output = builder.build_json(indent=2)
```

### RepoSnapshot v1 Usage (v0.4.0+)

**NEW**: Deterministic, version-controlled snapshots with runtime detection.

```python
from pathlib import Path
from intentgraph.snapshot import RepoSnapshotBuilder

# Generate deterministic snapshot
builder = RepoSnapshotBuilder(Path.cwd())
snapshot = builder.build()

# Access structure (80 files analyzed, includes tests by default)
print(f"Files: {len(snapshot.structure.files)}")
print(f"Languages: {[lang.language for lang in snapshot.structure.languages]}")

# Access runtime detection (no code execution!)
print(f"Package Manager: {snapshot.runtime.package_manager}")
print(f"Python Version: {snapshot.runtime.python_version}")
print(f"Tooling: {snapshot.runtime.tooling}")

# Serialize to JSON
json_output = builder.build_json(indent=2)
Path("snapshot.json").write_text(json_output)
```

**Key Features**:
- **Deterministic UUIDs**: SHA256-based, same path → same UUID
- **Stable Ordering**: All arrays sorted (reproducible output)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Raytracer76/IntentGraph](https://github.com/Raytracer76/IntentGraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
