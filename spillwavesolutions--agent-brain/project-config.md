---
trigger: always_on
description: Instructions for AI agents (Cursor, Windsurf, OpenCode, etc.) when working on this repository.
---


# Agent Brain Agent Guidelines

Instructions for AI agents (Cursor, Windsurf, OpenCode, etc.) when working on this repository.

Planning rule: after any planning step, save the plan to `docs/plans/<name>.md` before doing work.

## Project Overview

Agent Brain is a RAG-based document indexing and semantic search system. It's a monorepo containing:

| Package | Path | Description |
|---------|------|-------------|
| agent-brain-server | `agent-brain-server/` | FastAPI REST API server |
| agent-brain-cli | `agent-brain-cli/` | CLI management tool |
| agent-brain-skill | `agent-brain-skill/` | Claude Code skill |

## Technology Stack

- **Python**: 3.10+
- **Build System**: Poetry
- **Package Installer**: uv (preferred over pip)
- **Server**: FastAPI + Uvicorn
- **CLI**: Click + Rich
- **Vector Store**: ChromaDB
- **Embeddings**: OpenAI text-embedding-3-large
- **Indexing**: LlamaIndex

## Package Installation

**IMPORTANT**: Always use `uv` instead of `pip` for installing packages. It's faster and handles dependencies better.

```bash
# Build packages
cd agent-brain-server && poetry build
cd agent-brain-cli && poetry build

# Install with uv (NOT pip)
uv pip install dist/agent_brain_rag-*.whl --force-reinstall
uv pip install dist/agent_brain_cli-*.whl --force-reinstall
```

## Build and Test Commands

### agent-brain-server

```bash
cd agent-brain-server
poetry install                    # Install dependencies
poetry run agent-brain-serve      # Run server
poetry run pytest                 # Run tests
poetry run pytest --cov=agent_brain_server       # Tests with coverage
poetry run mypy agent_brain_server               # Type checking
poetry run ruff check agent_brain_server         # Linting
poetry run black agent_brain_server              # Format code
```

### agent-brain-cli

```bash
cd agent-brain-cli
poetry install                    # Install dependencies
poetry run agent-brain --help     # Show CLI help
poetry run pytest                 # Run tests
poetry run mypy agent_brain_cli               # Type checking
poetry run ruff check agent_brain_cli         # Linting
poetry run black agent_brain_cli              # Format code
```

### Full Quality Check

```bash
# Run from package directory
poetry run black agent_brain_server tests && poetry run ruff check agent_brain_server tests && poetry run mypy agent_brain_server && poetry run pytest
```

## Project Structure

```
doc-serve/
├── agent-brain-server/           # FastAPI server
│   ├── agent_brain_server/           # FastAPI server
│   │   ├── api/                # REST endpoints
│   │   │   ├── main.py         # App entry point
│   │   │   └── routers/        # Route handlers
│   │   ├── config/             # Settings (Pydantic)
│   │   ├── indexing/           # Document processing
│   │   ├── models/             # Request/response models
│   │   ├── services/           # Business logic
│   │   └── storage/            # ChromaDB integration
│   └── tests/
├── agent-brain-cli/                # CLI tool
│   ├── agent_brain_cli/           # CLI package
│   │   ├── cli.py              # Main entry point
│   │   ├── client/             # API client
│   │   └── commands/           # CLI commands
│   └── tests/
├── agent-brain-skill/            # Claude skill
│   └── doc-serve/
│       └── SKILL.md
└── docs/                       # Documentation
```

## Code Style

### Python Standards
- **Formatter**: Black (line length 88)
- **Linter**: Ruff
- **Type Checker**: mypy (strict mode)
- **Type Hints**: Required for all function signatures

### Style Guidelines
1. Use Google-style docstrings
2. Sort imports with Ruff/isort
3. Type hint all function parameters and returns
4. Keep functions focused and testable

## API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/health` | GET | Health check |
| `/health/status` | GET | Indexing status |
| `/query` | POST | Semantic search |
| `/query/count` | GET | Document count |
| `/index` | POST | Start indexing |
| `/index/add` | POST | Add documents |
| `/index` | DELETE | Clear index |

## CLI Commands

### Instance Commands

| Command | Description |
|---------|-------------|
| `agent-brain init` | Initialize project for Agent Brain |
| `agent-brain start` | Start Agent Brain server |
| `agent-brain stop` | Stop the running server |
| `agent-brain list` | List all running instances |

### Data Commands

| Command | Description |
|---------|-------------|
| `agent-brain status` | Check server status |
| `agent-brain query "text"` | Search documents |
| `agent-brain index /path` | Index documents (queued) |
| `agent-brain index /path --force` | Index, bypass deduplication |
| `agent-brain reset --yes` | Clear index |

### Job Queue Commands

| Command | Description |
|---------|-------------|
| `agent-brain jobs` | List all jobs in queue |
| `agent-brain jobs --watch` | Watch queue with live updates |
| `agent-brain jobs JOB_ID` | Show job details |
| `agent-brain jobs JOB_ID --cancel` | Cancel a job |

## Environment Variables

### Server (agent-brain-server/.env)

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `OPENAI_API_KEY` | Yes | - | OpenAI API key for embeddings |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SpillwaveSolutions/agent-brain](https://github.com/SpillwaveSolutions/agent-brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
