---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Academia MCP is an MCP (Model Context Protocol) server that provides tools for searching, fetching, analyzing, and reporting on scientific papers and datasets. It integrates with multiple academic APIs (ArXiv, ACL Anthology, Semantic Scholar, Hugging Face) and web search providers (Exa, Brave, Tavily), plus optional LLM-powered document analysis tools.

**Key Features:**
- ArXiv and ACL Anthology search/download
- Semantic Scholar citation graphs
- Hugging Face datasets search
- Web search and page crawling
- LaTeX compilation and PDF reading
- LLM-powered document QA and research proposal workflows

**Tech Stack:**
- Python 3.12+ with type hints (strict mypy)
- FastMCP framework for the MCP server
- OpenAI SDK for LLM calls (via OpenRouter)
- Pydantic for data models and settings
- Fire for CLI argument parsing
- Multiple transport options: stdio, SSE, streamable-http

## Development Commands

**IMPORTANT: Always prefer `make` commands when available.** The Makefile provides consistent, tested workflows.

### Setup
```bash
# Create virtual environment and install dependencies
uv venv .venv
make install
```

### Validation (ALWAYS run before committing)
```bash
# Format code with black (line length: 100)
make black

# Run all validation: black, flake8, mypy --strict
make validate
```

This is the most important command - run `make validate` frequently during development.

### Testing
```bash
# Run full test suite (via make)
make test

# Run a single test file
uv run pytest -s ./tests/test_arxiv_search.py

# Run a specific test
uv run pytest -s ./tests/test_arxiv_search.py::test_arxiv_search
```

### Running the Server Locally
```bash
# Run with streamable-http (default, port 5056)
uv run -m academia_mcp --transport streamable-http

# Run with stdio (for Claude Desktop)
uv run -m academia_mcp --transport stdio

# Run with custom port
uv run -m academia_mcp --transport streamable-http --port 8080
```

### Publishing
```bash
make publish  # Builds and publishes to PyPI
```

## Architecture

### Server Initialization (server.py)

The `create_server()` function in `academia_mcp/server.py` is the heart of the application:

1. **Core Tools** (always available): arxiv_search, arxiv_download, anthology_search, s2_* (Semantic Scholar), hf_datasets_search, visit_webpage, get_latex_templates_list, show_image, yt_transcript

2. **Conditional Tool Registration** (based on environment variables):
   - `WORKSPACE_DIR` set → enables compile_latex, download_pdf_paper, read_pdf
   - `OPENROUTER_API_KEY` set → enables LLM tools (document_qa, review_pdf_paper, bitflip tools, describe_image)
   - `EXA_API_KEY`/`BRAVE_API_KEY`/`TAVILY_API_KEY` set → enables respective web_search tools

3. **Transport Modes**:
   - `stdio`: for local MCP clients (Claude Desktop)
   - `streamable-http`: HTTP with CORS enabled for browser clients
   - `sse`: server-sent events

### Tool Structure

All tools live in `academia_mcp/tools/` and follow this pattern:
- Each tool is a standalone async function with type hints
- Tools use Pydantic models for inputs/outputs (enables structured_output mode)
- Most tools are registered with `structured_output=True` for schema validation
- Tools import from shared utilities (`utils.py`, `llm.py`, `settings.py`)

**Key Tool Categories:**
- **Search tools**: arxiv_search.py, anthology_search.py, s2.py, hf_datasets_search.py, web_search.py
- **Fetch/download tools**: arxiv_download.py, visit_webpage.py, review.py
- **Document processing**: latex.py (compile_latex, read_pdf), image_processing.py
- **LLM-powered tools**: document_qa.py, bitflip.py (research proposals), review.py

### Settings Management (settings.py)

Uses `pydantic-settings` to load configuration from `.env` file or environment variables:
- API keys: OPENROUTER_API_KEY, TAVILY_API_KEY, EXA_API_KEY, BRAVE_API_KEY, OPENAI_API_KEY
- Model names: REVIEW_MODEL_NAME, BITFLIP_MODEL_NAME, DOCUMENT_QA_MODEL_NAME, DESCRIBE_IMAGE_MODEL_NAME
- Workspace: WORKSPACE_DIR (Path), PORT (int)
- Authentication: ENABLE_AUTH (bool, default False), TOKENS_FILE (Path, default ./tokens.json)
- All settings accessible via `from academia_mcp.settings import settings`

### Authentication System (auth/)

The authentication system provides optional token-based security for HTTP transports (streamable-http, sse).

**Architecture:**
- Integrated authentication model (single server handles both token validation and MCP tools)
- Bearer token validation via Starlette middleware
- Token storage in JSON file with metadata (client_id, scopes, expiration, etc.)
- CLI commands for token lifecycle management

**Key Components:**

1. **Token Models** (`academia_mcp/auth/models.py`):
   - `TokenMetadata`: Stores token_id, client_id, scopes, issued_at, expires_at, description, revoked, last_used
   - `TokenStore`: Container for all tokens with version tracking
   - Token format: `mcp_<32 hex chars>` (128 bits of entropy via `secrets.token_hex(16)`)

2. **Token Manager** (`academia_mcp/auth/token_manager.py`):
   - `generate_token()`: Creates cryptographically secure tokens

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IlyaGusev/academia_mcp](https://github.com/IlyaGusev/academia_mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
