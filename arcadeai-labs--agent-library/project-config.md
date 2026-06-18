---
trigger: always_on
description: Handles modification time tracking for incremental updates
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Agent Library** (package name: `agent-library`, Python module: `librarian`) is a multi-modal knowledge library for AI agents built on Arcade for the Model Context Protocol (MCP). It provides persistent storage with semantic and keyword search for text, code, images, and PDFs.

**Naming note for contributors:** The product is "Agent Library." The Python module path is `librarian` (don't rename — it's the import surface). The MCP server identifies itself as `Librarian` (`MCPApp(name="Librarian")` in `server.py`), which is why exposed tools carry the `Librarian_` prefix. Keep that distinction in mind: prose says "Agent Library"; code-level identifiers say "librarian"/"Librarian".

### Key Technologies
- SQLite with `sqlite-vec` for vector search
- FTS5 with BM25 ranking for full-text search
- Hybrid search combining both approaches with Max Marginal Relevance (MMR)
- Configurable embedding models (local sentence-transformers or OpenAI-compatible API)
- Support for multi-modal assets (text, code, images, PDFs)

### Current Status
- **Multi-Modal Support Complete:** Code, PDF, and image parsing with asset type preservation
- **Parser Registry:** Automatic parser selection based on file extension
- **Database Schema:** Multi-modal columns (asset_type, modality_data) fully implemented
- **Search Integration:** All search tools return asset_type to AI agents
- See `IMPLEMENTATION_STATUS.md` for detailed progress tracking
- See `MULTI_MODAL_LIBRARIAN_DESIGN.md` for complete design specification

## Development Commands

### Setup & Installation
```bash
./setup.sh              # Initial setup
make install            # Install base dependencies
make sync               # Sync dependencies from pyproject.toml

# Install optional multi-modal dependencies
uv pip install -e ".[pdf]"      # PDF processing (pypdf)
uv pip install -e ".[vision]"   # Image processing (Pillow)
uv pip install -e ".[all]"      # All multi-modal features
```

**Optional Dependencies:**
- **PDF Support**: Install `pypdf` to enable PDF parsing
  ```bash
  uv pip install -e ".[pdf]"
  ```
  Without this, PDF files will be skipped during indexing.

- **Image Support**: Install `Pillow` to enable image metadata extraction
  ```bash
  uv pip install -e ".[vision]"
  ```
  Without this, image files will be skipped during indexing.

- **Code Parsing**: Works out-of-the-box with regex-based symbol extraction
  - No additional dependencies required
  - Supports 18+ programming languages

**Testing Multi-Modal Features:**
After installing optional dependencies, verify they work:
```bash
# Test PDF parsing
uv run pytest tests/test_multimodal.py::TestPDFParser -v

# Test image parsing
uv run pytest tests/test_multimodal.py::TestImageParser -v

# Test all multi-modal features
uv run pytest tests/test_multimodal.py -v
```

### Testing
```bash
make test               # Run all tests with coverage
make test-fast          # Run tests without coverage
uv run pytest tests/test_file.py  # Run specific test file
uv run pytest tests/test_file.py::TestClass::test_method  # Run specific test
uv run pytest -m slow   # Run slow tests (loads real embedding models)
```

### Code Quality
```bash
make check              # Run all checks (lint + format-check + typecheck)
make lint               # Run ruff linting
make lint-fix           # Auto-fix linting issues
make format             # Format code with ruff
make typecheck          # Run mypy type checking
```

### Building & Evaluation
```bash
make build              # Build wheel distribution
make evals              # Run Arcade tool evaluations
```

### CLI Usage
```bash
# Multi-modal indexing (auto-detects file types)
libr add ~/notes        # Index markdown files (AssetType.TEXT)
libr add ~/code         # Index source code (AssetType.CODE)
libr add ~/docs         # Index PDFs, images, code, markdown - all types
libr search "query"     # Search across all asset types

# Search results include asset_type
# Example: {"asset_type": "code", "document_path": "main.py", ...}

# Library management
libr list               # List all sources
libr index              # Show library statistics
libr serve stdio        # Start MCP server (stdio)
libr serve http --port 8000  # Start MCP server (HTTP)
```

## Architecture Overview

### Core Pipeline
**File → Parser → Chunker → Embedder → Database → Search**

### Type System
All components use strongly-typed enums:
- `AssetType`: TEXT, CODE, IMAGE, PDF, MULTIMODAL
- `SourceType`: DOCUMENTS, CODEBASE, KNOWLEDGE_BASE, ASSETS, MIXED
- `ChunkingStrategy`: HEADERS, PARAGRAPHS, SENTENCES, FIXED, CODE_BLOCKS, PAGES
- `EmbeddingModality`: TEXT, CODE, VISION, HYBRID
- `ProgrammingLanguage`: 18 supported languages (PYTHON, JAVASCRIPT, etc.)
- `CodeSymbolType`: FUNCTION, CLASS, METHOD, VARIABLE, etc.
- `SearchMode`: HYBRID, SEMANTIC, KEYWORD

### Document Processing (`librarian/processing/`)
1. **Parsers** (`parsers/`):
   - `base.py`: Base parser interface with parse_file() and parse_content()
   - `md.py`: Markdown parser with frontmatter (AssetType.TEXT)
   - `obsidian.py`: Obsidian-flavored markdown with wikilinks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arcadeai-labs/agent-library](https://github.com/arcadeai-labs/agent-library) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
