---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Model Context Protocol (MCP) server for Donetick chores management. It enables Claude and other MCP-compatible AI assistants to interact with a Donetick instance through a rate-limited API.

**Key Technologies:**
- Python 3.11+
- MCP SDK (>=1.20.0)
- httpx for async HTTP
- Pydantic for data validation
- Docker for containerization

## Project Structure

The project follows a clean directory structure:

```
donetick-mcp-server/
├── src/donetick_mcp/       # Source code
│   ├── server.py           # MCP server implementation
│   ├── client.py           # API client with rate limiting
│   ├── models.py           # Pydantic data models
│   └── config.py           # Configuration management
├── tests/                  # Formal test suite (pytest)
│   ├── test_client.py      # Unit tests for API client
│   └── test_server.py      # Integration tests for MCP server
├── tmp/                    # Temporary files (gitignored)
│   └── *.py                # Ad-hoc testing, analysis, verification scripts
├── .gitignore              # Excludes tmp/, venv/, build artifacts
└── pyproject.toml          # Project dependencies and metadata
```

**Important Conventions**:
- **Source code**: Always in `src/donetick_mcp/`
- **Formal tests**: Always in `tests/` (run via pytest)
- **Temporary files**: Always in `tmp/` for analysis, verification, one-off testing
- **Never commit**: Files in `tmp/` are gitignored - use for local experimentation only

When creating test scripts for verification or analysis, always place them in `tmp/` to keep the project root clean. The formal test suite in `tests/` should be reserved for permanent, repeatable tests that are part of CI/CD.

## Development Commands

### Setup & Installation

```bash
# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install for development
pip install -e ".[dev]"

# Install production only
pip install -e .
```

### Running the Server

```bash
# Run directly with Python
python -m donetick_mcp.server

# Or use entry point
donetick-mcp

# Run with Docker
docker-compose up -d

# View Docker logs
docker-compose logs -f donetick-mcp

# Stop Docker
docker-compose down
```

### Testing

```bash
# Run all tests
pytest

# Run with verbose output
pytest -v

# Run specific test file
pytest tests/test_client.py
pytest tests/test_server.py

# Run with coverage
pytest --cov=donetick_mcp --cov-report=html

# Run single test
pytest tests/test_client.py::test_list_chores -v
```

### Linting & Formatting

```bash
# Check code with ruff
ruff check src/

# Format code with ruff
ruff format src/
```

## Architecture

### High-Level Structure

The codebase follows a clean separation of concerns:

```
┌─────────────────┐
│   MCP Server    │ ← server.py: Exposes 5 tools to Claude
│   (server.py)   │   Handles tool registration & execution
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│  Donetick API   │ ← client.py: HTTP client wrapper
│  Client         │   Rate limiting, retry logic, auth
│  (client.py)    │   Connection pooling
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│  Pydantic       │ ← models.py: Type-safe data models
│  Models         │   Request/response validation
│  (models.py)    │
└─────────────────┘
```

### Key Components

#### 1. MCP Server (server.py)

- **Purpose**: Exposes Donetick functionality as MCP tools
- **Transport**: stdio (for Claude Desktop integration)
- **Global State**: Maintains a single `DonetickClient` instance
- **Tools Exposed (20 tools)**:
  - **Chore Management (10 tools)**:
    - `list_chores`: List with filters (active status, assigned user), supports `detail_level` for brief/full responses
    - `get_chore`: Get by ID (uses direct GET endpoint, includes sub-tasks)
    - `create_chore`: Create new chore (supports sub-tasks)
    - `complete_chore`: Mark complete (Premium feature)
    - `update_chore`: Update basic fields (name, description, due date)
    - `update_chore_priority`: Update priority level (0-4)
    - `update_chore_assignee`: Reassign chore to different user
    - `delete_chore`: Delete chore (creator only)
    - `skip_chore`: Skip chore without marking complete (reschedule next occurrence)
    - `update_subtask_completion`: Mark individual subtasks complete/incomplete with progress tracking
  - **Label Management (4 tools)**:
    - `list_labels`: List all labels in the circle
    - `create_label`: Create new custom label
    - `update_label`: Modify existing label
    - `delete_label`: Remove label
  - **Circle/User Management (3 tools)**:
    - `get_circle_members`: Get circle members with roles and points
    - `list_circle_users`: List all users in the circle
    - `get_user_profile`: Get current user's detailed profile
  - **History/Analytics (3 tools)**:
    - `get_chore_history`: Get completion history for a specific chore
    - `get_all_chores_history`: Get completion history across all chores (with pagination)
    - `get_chore_details`: Get detailed statistics and analytics for a chore


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jason1365/donetick-mcp-server](https://github.com/jason1365/donetick-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
