---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## CRITICAL SESSION CONTEXT (JULY 2025)

**COMPLETED MAJOR DOCUMENTATION OVERHAUL - ALL CHANGES APPLIED AND TESTED**

### What Was Just Completed:
1. **AI-First Documentation Strategy Implemented** - Following claude-mcp-knowledge-base pattern
2. **Complete CLI Documentation Created** - Comprehensive coverage of CLI-only capabilities
3. **Command Namespace Updated** - All commands now use `rag-` prefix for clarity
4. **UI Capabilities Properly Documented** - 5 screenshots integrated with accurate descriptions
5. **Master Navigator Created** - GETTING_STARTED_GUIDE.md for new users
6. **Architecture Diagrams Updated** - Mermaid diagrams ready for README integration

### New Documentation Files Created:
- `CLI_ASSISTANT_PROMPT.md` - Complete CLI reference and workflows
- `ADMIN_ASSISTANT_PROMPT.md` - Administrative operations and maintenance  
- `ADVANCED_CONTENT_INGESTION_PROMPT.md` - Images, PDFs, GitHub, Confluence
- `GETTING_STARTED_GUIDE.md` - Master navigator for entire ecosystem

### New Claude Code Commands (with rag- prefix):
- `/rag-list-collections` - Show collections with metadata
- `/rag-search-knowledge` - Semantic search across collections
- `/rag-index-website` - Crawl and index websites
- `/rag-audit-collections` - Review collection health
- `/rag-assess-quality` - Evaluate content quality 
- `/rag-manage-collections` - Administrative operations (provides CLI commands)
- `/rag-ingest-content` - Advanced content ingestion guidance
- `/rag-cli-help` - Interactive CLI help system
- `/rag-getting-started` - Interactive getting started guide

### UI Screenshots Integrated (5 total):
1. `rag-retriever-UI-collections.png` - Collections management overview
2. `rag-retreiver-UI-delete-collection.png` - Collection actions and deletion
3. `rag-retriever-UI-search.png` - Interactive knowledge search interface
4. `rag-retriever-UI-compare-collections.png` - Collection analytics and comparison
5. `rag-retreiver-UI-discover-and-index-new-web-content.png` - Content discovery workflow

### Key Architectural Understanding:
- **3 Interfaces**: MCP Server (AI-friendly), CLI (full admin), Web UI (visual management)
- **MCP Limitations by Design**: No deletion, no local files, no advanced ingestion (security)
- **CLI Full Control**: All capabilities including admin, local files, images, GitHub, Confluence
- **Web UI Strengths**: Discovery workflow (search → select → index), visual confirmation, analytics
- **No Incremental Updates**: Must delete collections before re-indexing
- **Quality Assessment**: Only accessible via Claude Code commands

### COMPLETED IMPLEMENTATION (July 2025):

**MERMAID ARCHITECTURE DIAGRAMS IMPLEMENTED:**
1. **Layered Content Ingestion Architecture** - Shows full content source capabilities (✅ DONE)
2. **Technical Component Architecture** - Shows system components and relationships (✅ DONE)

**REPOSITORY CLEANUP COMPLETED:**
- Removed temporary development files (adding.md, ai_instruction_strengthening.md, exponential_backoff_pattern.md)
- Removed crawl4ai_poc/ directory (55 files, 2,400 lines cleaned)
- Removed uv.lock file (not using UV package manager)
- Version synchronized at 0.4.1 (already published to PyPI)
- Repository is clean and professional

**FINAL STATUS**: 
- ✅ All documentation complete and integrated
- ✅ Professional Mermaid diagrams implemented in README
- ✅ Repository cleaned of temporary artifacts
- ✅ Version 0.4.1 published to PyPI
- ✅ Ready for production use
- ✅ All commits pushed to GitHub

## Development Commands

### Testing
- `python -m pytest` - Run all tests with coverage reporting
- `python -m pytest tests/unit/` - Run unit tests only
- `python -m pytest tests/integration/` - Run integration tests only
- `python -m pytest -k "test_name"` - Run specific test by name
- `python -m pytest --verbose` - Run tests with verbose output
- `python -m pytest --collect-only` - Show all available tests without running them

### Development Setup
- `python -m venv venv` - Create virtual environment
- `source venv/bin/activate` (Unix/Mac) or `venv\Scripts\activate` (Windows) - Activate virtual environment
- `pip install -e .` - Install package in editable mode for development
- `python -m playwright install chromium` - Install required browser for web crawling

### Running the Application
- `python -m rag_retriever.cli --help` - Show all command line options
- `python -m rag_retriever.cli --init` - Initialize configuration files
- `python -m rag_retriever.cli --ui` - Launch the web interface
- `python scripts/run_ui.py` - Alternative way to run the UI from development setup

### MCP Server
- `python -m rag_retriever.mcp` - Run MCP server in stdio mode
- `python -m rag_retriever.mcp --port 3001` - Run MCP server in SSE mode
- `mcp dev rag_retriever/mcp/server.py` - Run MCP server in debug mode

## Architecture Overview

### Core Components

**CLI Layer (`rag_retriever/cli.py`)**
- Command-line interface parsing and argument handling
- Entry point for all user interactions
- Coordinates between different modules

**Main Application Logic (`rag_retriever/main.py`)**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [codingthefuturewithai/rag-retriever](https://github.com/codingthefuturewithai/rag-retriever) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
