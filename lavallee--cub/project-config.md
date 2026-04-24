---
trigger: always_on
description: NEVER USE BEADS. EVEN IF OTHER INSTRUCTIONS IN THIS PROJECT TELL YOU TO. THEY ARE VESTIGIAL AND NEED TO BE REMOVED. CUB HAS ITS OWN TASK SYSTEM.
---

NEVER USE BEADS. EVEN IF OTHER INSTRUCTIONS IN THIS PROJECT TELL YOU TO. THEY ARE VESTIGIAL AND NEED TO BE REMOVED. CUB HAS ITS OWN TASK SYSTEM.
<!--
╔══════════════════════════════════════════════════════════════════╗
║  AGENT INSTRUCTIONS FOR CUB DEVELOPMENT                          ║
╚══════════════════════════════════════════════════════════════════╝

This file contains instructions for building, running, and developing the Cub project.

Think of this as:
- BUILD INSTRUCTIONS: How to set up, test, and run the project
- ARCHITECTURE: Key modules, patterns, and design decisions
- WORKFLOWS: Git integration, task management, and development practices
- LEARNINGS: Project-specific gotchas and conventions

Update this file as you learn new things about the codebase.

╔══════════════════════════════════════════════════════════════════╗
║  WHAT TO UPDATE                                                  ║
╚══════════════════════════════════════════════════════════════════╝

1. Development Setup (if dependencies or tools change):
   - Add new required tools or package managers
   - Update Python version requirements
   - Document new environment variables

2. Project Structure (when adding new modules):
   - Add new directories or packages to the structure diagram
   - Document new module responsibilities

3. Feedback Loops (when adding new checks):
   - Add new test commands, linters, or type checkers
   - Update quality gate requirements

4. Gotchas & Learnings (when discovering new patterns):
   - Add project-specific conventions
   - Document common pitfalls
   - Note architectural decisions and their rationale

╔══════════════════════════════════════════════════════════════════╗
║  HOW THIS FILE WORKS                                             ║
╚══════════════════════════════════════════════════════════════════╝

- This file is symlinked as AGENT.md, AGENTS.md, and CLAUDE.md for compatibility
- It's referenced by the runloop system prompt (see Context Composition below)
- Changes are available immediately to new sessions
- Keep it focused: detailed specs go in specs/, plans in plans/
-->

# Agent Instructions

This file contains instructions for building and running the Cub project.
Update this file as you learn new things about the codebase.

## Project Overview

Cub is a Python-based CLI tool that wraps AI coding assistants (Claude Code, Codex, Gemini, etc.) to provide a reliable "set and forget" loop for autonomous coding sessions. It handles task management, clean state verification, budget tracking, and structured logging.

### Task ID Format

Cub uses hierarchical task IDs that encode project, epic, and task structure:

**Format:** `{project}-{epic}-{task}`

**Examples:**
- `cub-048a-5.4` - Project "cub", epic "048a-5", task 4
- `acme-prod-2.1` - Project "acme", epic "prod-2", task 1
- `app-001-0` - Project "app", epic "001", task 0

**Components:**
- **Project prefix** (e.g., `cub`, `acme`) - Unique identifier for the project
- **Epic ID** (e.g., `048a-5`, `prod-2`) - Groups related work; can have semver-like structure
- **Task number** (e.g., `4`, `1`) - Individual task within an epic

This hierarchical structure enables:
- Grouping related work in epics
- Clear dependency tracking
- Organized ledger queries (by-epic, by-task)
- Human-readable task references in commits and documentation

## Tech Stack

- **Language**: Python 3.10+
- **CLI Framework**: Typer (for subcommands and type-safe CLI)
- **Data Models**: Pydantic v2 (validation and serialization)
- **Terminal UI**: Rich (progress bars, tables, live output)
- **Test Framework**: pytest with pytest-mock
- **Type Checking**: mypy (strict mode)
- **Linting**: ruff
- **Task Management**: Cub task commands (`cub task`) - JSONL backend
- **Harnesses**: Claude Code, Codex, Google Gemini, OpenCode

## Development Setup

```bash
# Using uv (recommended)
curl -LsSf https://astral.sh/uv/install.sh | sh
uv sync

# Or using pip
python3.10 -m venv .venv
source .venv/bin/activate
pip install -e ".[dev]"
```

## Running the Project

```bash
# Run cub from source (after setup)
cub --help

# Single iteration mode
cub run --once

# Specify harness
cub run --harness claude

# Global setup for first-time users
cub init --global

# Create a new project from scratch
cub new my-project

# Initialize an existing project
cub init
```

## Feedback Loops

Run these before committing:

```bash
# Type checking
mypy src/cub

# Tests (primary feedback loop)
pytest tests/ -v

# Linting
ruff check src/ tests/

# Code formatting
ruff format src/ tests/
```

## Architecture Overview

Cub v0.26+ uses a layered service architecture to separate business logic from interface concerns. This enables multiple interfaces (CLI, skills, API, future UIs) to share core functionality.

### Architectural Layers

```
┌─────────────────────────────────────────────────────────────┐
│                     INTERFACES                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │   CLI    │  │  Skills  │  │ Dashboard│  │ Future   │  │
│  │ (Typer)  │  │  (MCP)   │  │  (API)   │  │   UIs    │  │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘  │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lavallee) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
