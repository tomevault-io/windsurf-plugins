---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**MGCP** (Memory Graph Core Primitives) is a Python MCP server providing persistent, graph-based memory for LLM interactions. The system stores lessons learned during LLM sessions in a graph structure, allowing semantic querying without loading full context histories.

**Status**: v2.1.0 - Alpha/Research project. Phases 1-7 complete, actively dogfooding. Phase 8 (skill compilation) removed — degraded reliability.

## Documentation Preferences

When creating diagrams, charts, tables, or other visuals, use HTML documents with visualization libraries:
- **Diagrams/Flowcharts**: mermaid.js
- **Charts/Graphs**: chart.js
- **Tables**: HTML tables with CSS styling

Avoid ASCII art diagrams in documentation. HTML visuals are easier to digest for a wider audience.

## Technology Stack

- Python 3.11+ with virtual environment (`.venv/`)
- FastMCP for MCP server framework
- NetworkX for graph operations
- Qdrant for vector storage (lessons + catalogue + workflows)
- sentence-transformers for local embeddings (`BAAI/bge-base-en-v1.5`, 768 dimensions)
- Pydantic for data validation
- SQLite + JSON for persistence
- FastAPI for web dashboard

## Development Commands

```bash
# Run the MCP server
python -m mgcp.server

# Run the web UI server
python -m mgcp.web_server

# Run tests
pytest

# Run a single test
pytest tests/test_basic.py::test_name

# Run linter
ruff check src/
```

## Data Management Commands

```bash
# Backup MGCP data
mgcp-backup                          # Create backup in current directory
mgcp-backup -o backup.tar.gz         # Specify output file
mgcp-backup --list                   # Preview what would be backed up
mgcp-backup --restore backup.tar.gz  # Restore from backup

# Export/Import lessons
mgcp-export lessons -o lessons.json  # Export lessons to JSON
mgcp-export projects -o proj.json    # Export project contexts
mgcp-import lessons.json             # Import lessons (skips duplicates)
mgcp-import data.json --merge overwrite  # Overwrite duplicates
mgcp-import data.json --dry-run      # Preview import without changes

# Find duplicate lessons
mgcp-duplicates                      # Find similar lessons (0.85 threshold)
mgcp-duplicates -t 0.90              # Higher threshold for stricter matching

# Bootstrap lessons and workflows
mgcp-bootstrap                       # Seed all (core + dev)
mgcp-bootstrap --update-triggers     # Update trigger fields on existing lessons

# Migrate from ChromaDB to Qdrant (for existing installations)
mgcp-migrate                         # Migrate data to Qdrant
mgcp-migrate --dry-run               # Preview what would be migrated
mgcp-migrate --force                 # Overwrite existing Qdrant data
```

## Architecture

The system flows from Claude/LLM through MCP Protocol to the MGCP Server, which contains Query Handler, Lesson Manager, and Graph Walker components. These connect to dual storage backends: Graph Store (NetworkX) and Vector Store (Qdrant).

### Core Components

All source files are in `src/mgcp/`:

- `server.py` - MCP server with 37 tools
- `models.py` - Pydantic models (Lesson, ProjectContext, ProjectCatalogue, SecurityNote, Convention, etc.)
- `graph.py` - NetworkX graph operations with typed relationships and Louvain community detection
- `embedding.py` - Centralized BGE embedding model (`BAAI/bge-base-en-v1.5`)
- `qdrant_vector_store.py` - Qdrant integration for lesson, workflow, and community summary search
- `qdrant_catalogue_store.py` - Qdrant integration for project catalogue search
- `persistence.py` - SQLite/JSON storage for lessons, project contexts, and community summaries
- `telemetry.py` - Usage tracking and analytics
- `web_server.py` - FastAPI web UI for browsing lessons/projects
- `launcher.py` - Unified CLI launcher
- `bootstrap.py` - Initial lesson seeding
- `migration.py` - ChromaDB to Qdrant migration tool
- `migrations.py` - Database migrations
- `init_project.py` - Multi-client MCP configuration (8 LLM clients supported)
- `data_ops.py` - Export, import, and duplicate detection
- `rem_cycle.py` - REM (Recalibrate Everything in Memory) cycle engine
- `rem_config.py` - REM scheduling strategies (linear, fibonacci, logarithmic)
- `backup.py` - Backup and restore functionality
- `bootstrap_loader.py` - Load bootstrap lessons, workflows, and relationships from YAML files
- `logging_config.py` - Centralized logging with rotation (10MB max, 5 backups)
- `reminder_state.py` - Self-directed reminder system for LLM workflow continuity

### Data Model

**Lessons** have hierarchical relationships (parent/child) and typed cross-links. Key fields:
- `trigger`: When the lesson applies (keywords/patterns)
- `action`: What to do (imperative)
- `tags`: Categorization for retrieval

**Project Contexts** persist across sessions with:
- Todos with status tracking
- Active files being worked on
- Recent decisions
- Notes about current state

**Project Catalogues** store project-specific knowledge:
- Architecture notes and gotchas
- Security notes with severity/status
- Conventions (naming, style, structure)
- File couplings (files that change together)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/devnullnoop) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-15 -->
