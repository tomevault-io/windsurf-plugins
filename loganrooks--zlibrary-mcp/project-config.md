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

## GSD Model Profile

**IMPORTANT:** Before spawning any GSD subagent, read `.planning/config.json` and use the `model_profile` value to select the correct model. Never hardcode or assume the profile.

| Agent | quality | balanced | budget |
|-------|---------|----------|--------|
| gsd-phase-researcher | opus | sonnet | haiku |
| gsd-planner | opus | opus | sonnet |
| gsd-plan-checker | sonnet | sonnet | haiku |
| gsd-executor | opus | sonnet | sonnet |

## 🚀 Quick Start for Claude Code

**Essential Reading Order**:
1. `.claude/PROJECT_CONTEXT.md` - Complete project understanding (mission, domain)
2. `.claude/ROADMAP.md` - **Strategic plan** (1-3 weeks, current priorities)
3. `.claude/ARCHITECTURE.md` - **System structure** (components, design decisions, status)
4. `ISSUES.md` - Known problems and priorities (at project root)
5. `.claude/IMPLEMENTATION_ROADMAP.md` - Concrete action plan and fixes
6. `.claude/PATTERNS.md` - Code patterns to follow
7. `.claude/RAG_QUALITY_FRAMEWORK.md` - Quality verification for RAG pipeline
8. `.claude/TDD_WORKFLOW.md` - Rigorous real-world testing process
9. `.claude/DEBUGGING.md` - Troubleshooting guide
10. `.claude/VERSION_CONTROL.md` - Git workflow and best practices
11. `.claude/CI_CD.md` - CI/CD strategy and implementation
12. `.claude/MCP_SERVERS.md` - Development tools setup
13. `.claude/META_LEARNING.md` - Lessons learned and insights

**Session Resumption** (for AI assistants):
- **Start**: Use `/sc:load` to restore context from Serena memory
- **Orient**: Review ROADMAP.md (strategic direction) → ARCHITECTURE.md (system state)
- **Work**: Follow TDD_WORKFLOW.md for RAG features, PATTERNS.md for code
- **End**: Use `/sc:save` to preserve session state in Serena memory

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

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [loganrooks/zlibrary-mcp](https://github.com/loganrooks/zlibrary-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
