---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Destructive Operations Policy

**MANDATORY before any destructive git operation** (`git filter-repo`, `git rm`, `git reset --hard`, force-push, history rewriting, bulk file deletion/archival):

1. **Dependency audit**: Search the entire codebase for references to affected files/paths (`grep -r`, check imports, configs, test fixtures, scripts)
2. **Present findings**: Show the user what depends on the targets and what will break
3. **Get explicit approval**: Do NOT proceed until the user confirms after seeing the dependency analysis
4. **One step at a time**: Never chain destructive operations — commit and verify between each

This applies equally to "cleanup" tasks. A file that looks stale may be a test fixture, a script input, or referenced by documentation that matters. Always check before removing.

## 🚀 Quick Start for Claude Code

**Essential Reading Order**:
0. `VISION.md` - Invariants and non-goals (what the project is, at project root)
1. `.claude/PROJECT_CONTEXT.md` - Complete project understanding (mission, domain)
2. `.claude/ARCHITECTURE.md` - **System structure** (components, design decisions, status)
3. `ISSUES.md` - Known problems and priorities (at project root)
4. `claudedocs/architecture/repo-health-and-roadmap-2026-07-24.md` - Health assessment and forward roadmap
5. `.claude/PATTERNS.md` - Code patterns to follow
6. `.claude/RAG_QUALITY_FRAMEWORK.md` - Quality verification for RAG pipeline
7. `.claude/TDD_WORKFLOW.md` - Rigorous real-world testing process
8. `.claude/DEBUGGING.md` - Troubleshooting guide
9. `.claude/VERSION_CONTROL.md` - Git workflow and best practices
10. `.claude/CI_CD.md` - CI/CD strategy and implementation

## Project Overview

Z-Library MCP (Model Context Protocol) server that enables AI assistants to search, download, and process books from Z-Library. The project uses a Node.js/TypeScript frontend with a Python bridge backend for document processing.

## Architecture

### Dual-Language Design
- **Node.js/TypeScript Layer** (`src/`): MCP server implementation handling tool registration and client communication
- **Python Bridge** (`lib/python_bridge.py`, `lib/rag_processing.py`): Handles Z-Library API interaction and document processing (EPUB, TXT, PDF)
- **Vendored Z-Library Fork** (`zlibrary/`): Modified fork of sertraline/zlibrary with custom download logic

### Key Components
- `src/index.ts`: MCP server entry point with tool definitions
- `src/lib/zlibrary-api.ts`: Bridge between Node.js and Python via PythonShell
- `src/lib/venv-manager.ts`: Manages Python virtual environment lifecycle
- `lib/python_bridge.py`: Core Python logic for Z-Library operations
- `lib/rag_processing.py`: Document processing for RAG workflows

### Data Flow
1. MCP client → Node.js server (tool request)
2. Node.js → Python bridge (via PythonShell)
3. Python → Z-Library API or document processing
4. Results flow back through the same chain

### Path Resolution Strategy

**Design Decision**: Python scripts remain in source `lib/` directory (not copied to `dist/`)

**Runtime Path Logic**:
```typescript
// From dist/lib/python-bridge.js at runtime:
const scriptPath = path.resolve(__dirname, '..', '..', 'lib', 'python_bridge.py');

// Navigation: dist/lib/ → dist/ → project_root/ → lib/python_bridge.py
```

**Path Helper Module** (Recommended for new code):
```typescript
import { getPythonScriptPath, getPythonLibDirectory } from './lib/paths.js';

const scriptPath = getPythonScriptPath('python_bridge.py');
// Returns: /project/lib/python_bridge.py

const libDir = getPythonLibDirectory();
// Returns: /project/lib
```

**Benefits**:
- ✅ Single source of truth (Python scripts in `lib/`)
- ✅ No build process changes needed
- ✅ No file duplication
- ✅ Development-friendly (edit Python directly)

**Validation**: Build automatically validates all Python files exist (`npm run build`)

**Documentation**: See [ADR-004](docs/adr/ADR-004-Python-Bridge-Path-Resolution.md) for complete rationale and [DEPLOYMENT.md](docs/DEPLOYMENT.md) for edge cases.

## Development Commands

### Setup (v2.0.0 - UV-based)
```bash
# Prerequisites: Install UV (one-time)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Initial setup (UV creates .venv and installs all dependencies)
bash setup-uv.sh
# Or manually: uv sync

# Install Node dependencies
npm install

# Build TypeScript
npm run build
```

**UV Migration (v2.0.0)**:
- Uses UV for Python dependency management (2025 best practice)
- Creates `.venv/` in project (portable, moves with project)
- Generates `uv.lock` for reproducible builds
- **Code Simplification**: 77% reduction (406 → 92 lines in venv-manager.ts)
- **Test Simplification**: 90% reduction (833 → 85 lines in tests)
- See `docs/MIGRATION_V2.md` for details

### Testing
```bash
# Run Jest (Node.js) tests — npm test does NOT run pytest
npm test

# Run Python tests (separate command)
uv run pytest
# Or: .venv/bin/python -m pytest

# Run specific Python test
uv run pytest __tests__/python/test_rag_processing.py::TestProcessDocumentForRAG::test_process_epub

# Run Jest tests with coverage

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rookslog/zlibrary-mcp](https://github.com/rookslog/zlibrary-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
