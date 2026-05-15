---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Journal

Keep development journal in @JOURNAL.md. Update it after each major change.

## Note-Taking Guidelines

- When asked to take notes: 1) Fill out @JOURNAL.md; 2) Add only the most important information to @CLAUDE.md 

## Development Setup

This project uses Python 3.13+ with `uv` as the package manager and `direnv` for environment management.

**Essential Commands:**
- `uv sync` - Install/update dependencies (automatically triggered by direnv)
- `uv add <package>` - Add a dependency
- `uv run pytest` - Run unit tests
- `uv run main.py` - Run the application
- `uv build` - Build package for distribution
- `uvx mcp-mcp` - Install and run via uvx (end-user command)
- `just update-readme` - Update server count in README.md (test after README changes)

**Environment Configuration:**
No special environment configuration required. Uses standard Python/uv project setup.

## Project Architecture

See @PRD.md for the product requirements and architecture documentation.

**MCP-MCP (Meta-MCP) Server** - A tool discovery and provisioning service for MCP servers that helps AI assistants dynamically find, containerize, and use MCP servers on-demand.

**Core Technology Stack:**
- Python 3.13+ with uv package manager
- Key dependencies: `httpx`, `mcp`, `sentence-transformers`, `rich`, `pydantic-settings`
- Planned: Docker for server containerization, GitHub API for discovery

**Development Status:**
1. **MVP Complete**: Multi-source discovery with semantic search and security hardening ✅
2. **Future Work**: Docker integration, MCP protocol proxy, advanced server management

**Current Architecture Patterns:**
- **Multi-Source Discovery**: Aggregate servers from 3 curated sources with intelligent deduplication
- **Semantic Search**: Use sentence-transformers for accurate capability matching
- **Security Middleware**: Origin validation preventing DNS rebinding attacks
- **Production Distribution**: Automated releases with precomputed data for fast startup

## Current Implementation Status

**MVP COMPLETE ✅:**
- **Multi-Source Discovery**: 3 curated sources providing comprehensive MCP server coverage
- **Semantic Search**: sentence-transformers with precomputed embeddings for sub-second search
- **Security Hardened**: Origin validation middleware preventing DNS rebinding attacks
- **Production Distribution**: uvx/pipx installation, Claude Desktop integration, automated releases
- **Test Architecture**: Comprehensive test suite with 65+ tests covering unit and integration scenarios
- **Documentation**: Complete README.md with security, development workflow, and user guides

**MVP Security Features ✅:**
- **Origin Validation Middleware**: Prevents DNS rebinding attacks by restricting Origins to localhost
- **Host Header Validation**: Validates Host headers to ensure requests come from allowed sources  
- **Production Hardening**: Only allows localhost/127.0.0.1 in production (no test hostnames)
- **Security Test Coverage**: AsyncMock-based tests ensuring middleware reliability

**Future Work (Beyond MVP):**
- Docker integration for automatic server containerization
- MCP protocol proxy for seamless server execution
- GitHub API integration for live server discovery
- Advanced server management and lifecycle features

**Main Entry Point:** `main.py` - full FastMCP server with CLI interface and security middleware

## MCP Protocol Integration

The server exposes this primary tool:
```python
def find_mcp_tool(description: str, example_question: str | None = None) -> dict:
    """Look for an MCP server that can provide the requested functionality.
    
    Returns server details with complete README documentation for setup instructions.
    """
```

**Discovery Sources (planned):**
1. Curated MCP server lists
2. GitHub repositories (search: `mcp language:python/javascript/go stars:>5`)
3. Package registries (npm, PyPI)
4. Direct Git repository URLs

## Schema Versioning

**Current Schema**: v1.0 with fields: `name`, `description`, `url`, `category`, `source`
**Data Size**: ~2MB (small enough for fast fallback)

**Simple Strategy**:
- ✅ v1.x data: Use precomputed data (instant startup)
- ❌ v2.x+ data: Fall back to live GitHub sources (3-second startup)

**Safety Guarantees (Tested & Verified)**: 
- **Never breaks user process**: v1.x client + v2.x data = automatic fallback (no crash)
- **Graceful degradation**: Malformed entries are skipped, process continues  
- **Always functional**: Live GitHub sources ensure system always works
- **Multiple safety layers**: Schema check → Data validation → Individual entry parsing → Exception handling
- **User experience**: Incompatible data = 3-second startup instead of instant (barely noticeable)

**Why This Works**: For our small data size, downloading fresh from GitHub is fast and reliable. No complex migration needed.

**Implementation**: `db/schema_versions.py` - simple compatibility check with fallback

## Security Considerations

- No API keys required for core functionality
- Docker containers planned for MCP server isolation
- No privileged container execution

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wojtyniak/mcp-mcp](https://github.com/wojtyniak/mcp-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
