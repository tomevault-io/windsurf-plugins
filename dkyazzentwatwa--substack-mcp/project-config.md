---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

```bash
# Environment setup
python3 -m venv .venv
source .venv/bin/activate
pip install -e .[dev]

# Testing
pytest                              # Run all tests
pytest tests/test_analysis.py       # Specific test module
pytest -v                           # Verbose output

# Testing the MCP server
./run_mcp_with_venv.sh             # Run MCP server for Claude Desktop
```

## Architecture Overview

This is a **Substack MCP (Model Context Protocol) Server** designed for use with Claude Code. It scrapes public Substack content and provides analytics through the MCP protocol.

### Core Components

1. **SubstackPublicClient** (`client.py`) - HTTP wrapper with rate limiting (1 second throttle) and TTL caching (15 minutes)

2. **Data Models** (`models.py`) - Pydantic schemas: `PostSummary`, `PostContent`, `AuthorProfile`, `Note`, `ContentAnalytics`

3. **Content Parsers** (`parsers.py`) - RSS/Atom feed parsing and HTML content extraction using BeautifulSoup

4. **Analytics Engine** (`analysis.py`) - VADER sentiment analysis, Flesch Reading Ease, keyword extraction (TF-IDF), publishing cadence

5. **MCP Server** (`server.py`) - MCP protocol implementation using stdio transport

### MCP Protocol Integration

The server implements the MCP protocol for Claude Desktop/Code integration:

**MCP Tools Available:**
- `search` - Basic search across publications
- `get_posts` - Fetch recent posts from a publication (limited to ~20)
- `get_all_posts` - **NEW**: Fetch ALL posts from publication archive with date filtering
- `search_substack` - Intelligent search with auto-discovery of relevant publications
- `get_content` - Smart content retrieval (URLs, publication names, handles)
- `discover_publications` - Discover publications by topic/industry
- `analyze_trends` - Analyze content trends and publishing patterns
- `get_post_content` - Full content of specific posts
- `analyze_post` - Sentiment and readability analysis
- `get_author_profile` - Get author and publication information
- `get_notes` - Fetch recent notes from a publication
- `search_notes` - Search notes by text content
- `crawl_publication` - Comprehensive publication intelligence

**Critical MCP Implementation Details:**
- MCP SDK locked to version 1.10.0 due to CallToolResult serialization bug in later versions
- Custom `create_text_result()` workaround returns plain dictionaries
- Stdio transport for Claude Desktop integration
- All tools return structured JSON responses

### Environment Configuration

```bash
# Optional configuration
SUBSTACK_MCP_THROTTLE=1.0          # Request throttling seconds (default: 1.0)
SUBSTACK_MCP_CACHE_TTL=900         # Cache TTL seconds (default: 900)
SUBSTACK_MCP_WARM_PUBLICATION      # Publication to warm cache on startup
```

### Package Structure

- Source code in `src/substack_mcp/` with setuptools configuration in `pyproject.toml`
- Editable install with `-e .` in `requirements.txt`
- MCP server wrapper script: `run_mcp_with_venv.sh`

### Key Implementation Patterns

**Error Handling:**
- HTTP status errors mapped to appropriate MCP error responses
- Graceful degradation when content unavailable
- Thread-safe caching with RLock

**Rate Limiting & Caching:**
- 1-second throttle between requests to respect Substack ToS
- TTL-based in-memory cache (15 minutes default)
- Public content only, no authentication bypass

**Data Flow:**
1. MCP request → Tool handler
2. Tool handler → SubstackPublicClient → HTTP request (throttled)
3. HTTP response → Parser → Pydantic model → Analytics (optional)
4. Result → MCP response format → Claude Desktop/Code

### Testing Strategy

- Unit tests with `pytest` and `pytest-asyncio`
- HTTP mocking with `respx` to avoid live network calls
- Sample fixtures for HTML/RSS content in `tests/fixtures/`
- Test coverage for all MCP tools

### Known Issues & Workarounds

1. **MCP SDK Bug**: Locked to version 1.10.0 due to CallToolResult serialization issues in later versions
2. **Field Mapping**: Fixed Pydantic field mismatches (`published` → `published_at`)
3. **JSON Schema**: Removed unsupported `default` properties from MCP tool schemas
4. **Stdio Transport**: MCP server uses stdio, so manual testing will appear to "hang" waiting for input (this is normal)

## Code Organization

### Client Layer (`client.py`)
- `SubstackPublicClient` - Main HTTP client
- Rate limiting: 1 request per second
- TTL cache: 15 minutes default
- Methods:
  - `fetch_feed()` - Recent posts from RSS feed (limit: ~20)
  - `fetch_archive()` - **NEW**: ALL posts from archive API with pagination and date filtering
  - `fetch_post()` - Full post content from URL
  - `fetch_author_profile()` - Author and publication metadata
  - `fetch_notes()` - Recent notes from publication
  - `search_notes()` - Search notes by text content
  - `crawl_publication()` - Comprehensive publication crawl

### Models (`models.py`)
All Pydantic v2 models with proper validation:
- `PostSummary` - Basic post metadata
- `PostContent` - Full post content with body
- `AuthorProfile` - Author and publication info
- `Note` - Substack notes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dkyazzentwatwa/substack_mcp](https://github.com/dkyazzentwatwa/substack_mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
