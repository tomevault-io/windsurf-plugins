---
trigger: always_on
description: This file provides guidance to AI coding agents (GitHub Copilot, Cursor, Aider, Claude Code, etc.) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents (GitHub Copilot, Cursor, Aider, Claude Code, etc.) when working with code in this repository.

> **Note:** For Claude Code specifically, see [CLAUDE.md](CLAUDE.md) which is optimized for Claude's interaction patterns.

## Project Overview

RivalSearchMCP is an MCP server providing 10 specialized tools for web research, content discovery, and analysis. Built on FastMCP, it offers multi-engine search (Yahoo/DuckDuckGo), website traversal, scientific research, and AI-enhanced research workflows. Ships with Claude Code Agent Skills for standalone CLI usage.

- **100% free**: No API keys required for core functionality
- **Dual deployment**: Hosted service at `https://RivalSearchMCP.fastmcp.app/mcp` or local development
- **Transport modes**: stdio (default, for CLI/IDE) or HTTP (production via `ENVIRONMENT=production`)

## Development Commands

### Setup & Running
```bash
# Install dependencies
pip install -r requirements.txt

# Install with dev dependencies
pip install -e ".[dev]"

# Set up pre-commit hooks
pre-commit install

# Run server (stdio mode - default for MCP clients)
python server.py

# Run in production mode (HTTP transport)
ENVIRONMENT=production python server.py

# Custom port
PORT=8080 ENVIRONMENT=production python server.py
```

### Testing
```bash
# Run all tests
pytest

# Run specific test file
pytest tests/test_client.py

# Run specific test function
pytest tests/test_client.py::test_function_name

# Run with coverage report
pytest --cov=src --cov-report=term-missing

# Filter by markers
pytest -m unit           # Only unit tests
pytest -m integration    # Only integration tests
pytest -m "not slow"     # Skip slow tests

# Verbose with short traceback
pytest -v --tb=short
```

### Code Quality
```bash
# Format code
black src/ tests/

# Sort imports
isort src/ tests/

# Lint with auto-fix
ruff check --fix src/ tests/

# Type checking
mypy src/

# Run all pre-commit hooks
pre-commit run --all-files
```

### Documentation
```bash
# Serve locally at http://127.0.0.1:8000
mkdocs serve

# Build static site
mkdocs build

# Deploy to GitHub Pages
mkdocs gh-deploy
```

## Architecture

### High-Level Structure

RivalSearchMCP uses a **modular tool-based architecture** where each tool category registers MCP tools with the FastMCP app in `server.py`. The architecture separates tool interfaces from core business logic:

```
src/
├── tools/          # MCP tool implementations (10 tools)
│   ├── analysis.py       # content_operations tool
│   ├── multi_search.py   # multi_search tool
│   ├── research.py       # research_topic tool
│   ├── scientific.py     # scientific_research tool
│   ├── search.py         # Tool registration orchestration
│   ├── traversal.py      # traverse_website tool
│   ├── trends.py         # trends_core, trends_export tools
│   └── research_modules/ # AI-enhanced research (OpenRouter integration)
├── core/           # Core business logic (reusable, not MCP-specific)
│   ├── search/     # Multi-engine search (DDG, Bing, Yahoo, Mojeek, Wikipedia)
│   ├── content/    # UnifiedContentExtractor
│   ├── news/       # 5-source news aggregator
│   ├── social/     # 9 platform adapters
│   ├── scientific/ # Academic (OpenAlex, CrossRef, arXiv, PubMed, Europe PMC)
│   │              # + datasets (Kaggle, HuggingFace, Dataverse, Zenodo)
│   ├── quality/    # Source-quality scoring
│   ├── conflict/   # Numeric / date / polarity conflict detection
│   ├── memory/     # Persistent research workspaces
│   ├── traverse/   # Website crawling logic
│   ├── cache/      # py-key-value-aio backed cache
│   └── security/   # Rate limiting, IP filtering
├── middleware/     # FastMCP middleware (timing, security, metrics)
├── schemas/        # Pydantic validation schemas
└── utils/          # Shared utilities (content, parsing, headers, etc.)
```

### Key Architectural Patterns

**Tool Registration Pattern:**
Each `src/tools/<category>.py` exports a `register_<category>_tools(app: FastMCP)` function that decorates tool implementations and registers them with the FastMCP app. All registration functions are called in `server.py` (lines 109-116).

Example pattern:
```python
def register_search_tools(app: FastMCP):
    @app.tool(name="multi_search", description="...")
    async def multi_search(query: str, ...) -> SearchResults:
        # Uses core modules from src/core/search/
        pass
```

**Multi-Engine Search Architecture:**
- Core implementation: `src/core/search/core/multi_engines.py`
- Concurrent execution across Yahoo and DuckDuckGo using asyncio
- Automatic fallback when individual engines fail
- Result deduplication by URL via `MultiSearchResult` class
- Intelligent merging of results from multiple sources

**Content Processing Pipeline:**
- 6-tier fallback system in `src/core/content/extractors.py`
- HTML → Markdown conversion via `src/utils/content.py::clean_html_to_markdown()`
- Parser preference: selectolax (fastest) > lxml > html.parser
- BeautifulSoup4 used for robustness with lxml backend

**AI Integration (Optional):**
- OpenRouter integration in `src/tools/research_modules/`
- Used by `research_workflow` tool for AI-enhanced research

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [damionrashford/RivalSearchMCP](https://github.com/damionrashford/RivalSearchMCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
