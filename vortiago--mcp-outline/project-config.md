---
trigger: always_on
description: This guide helps you implement and modify the MCP Outline server effectively.
---

# MCP Outline Server Guide

This guide helps you implement and modify the MCP Outline server effectively.

## Purpose

This MCP server bridges AI assistants with Outline's document management platform:
- REST API integration for Outline services
- Tools for documents, collections, attachments, and comments
- MCP resources via `outline://` URI scheme
- API key authentication with rate limiting
- Docker and local development support
- Health check endpoints for container orchestration

## Architecture

### Tool Categories

- **Search**: Find documents, collections, hierarchies
- **Reading**: Read content, export markdown
- **Attachments**: Resolve URLs, fetch content, list attachments
- **Content**: Create, update, comment (supports templates)
- **Organization**: Move documents between collections
- **Lifecycle**: Archive, delete, restore operations
- **Collaboration**: Comments, backlinks
- **Collections**: Create, update, delete, export
- **Batch Operations**: Bulk create, update, move, archive, delete
- **AI**: Natural language queries

### Feature Registration Flow

```
register_all(mcp)
  |- health.register_routes(mcp)               # Always
  |- documents.register(mcp)
  |   |- document_search.register_tools()      # Always
  |   |- document_reading.register_tools()     # Always
  |   |- document_attachments.register_tools() # Always
  |   |- document_collaboration.register_tools() # Always
  |   |- collection_tools.register_tools()     # Always (exports always, writes conditional)
  |   |- ai_tools.register_tools()             # If not OUTLINE_DISABLE_AI_TOOLS
  |   |- document_content.register_tools()     # If not OUTLINE_READ_ONLY
  |   |- document_lifecycle.register_tools()   # If not OUTLINE_READ_ONLY
  |   |- document_organization.register_tools() # If not OUTLINE_READ_ONLY
  |   |- batch_operations.register_tools()     # If not OUTLINE_READ_ONLY
  |- resources.register(mcp)                   # Always
install_dynamic_tool_list(mcp)                   # If OUTLINE_DYNAMIC_TOOL_LIST=true
```

For dynamic tool list architecture and scope matching details, see
[docs/dynamic-tool-list.md](docs/dynamic-tool-list.md).

### MCP Resources (`outline://` URI scheme)

- `outline://document/{document_id}` - Full markdown content
- `outline://document/{document_id}/backlinks` - Documents linking to this one
- `outline://collection/{collection_id}` - Collection metadata
- `outline://collection/{collection_id}/tree` - Hierarchical document tree
- `outline://collection/{collection_id}/documents` - List documents in collection

### Health Check Endpoints

- `GET /health` - Liveness probe (always returns 200)
- `GET /ready` - Readiness probe (verifies API connectivity, returns 503 if not ready)

## Core Concepts

### Outline Objects

- **Documents**: Markdown content with title, URL, and metadata
- **Collections**: Grouping with name, description, color
- **Comments**: Threaded discussions with replies and anchor text
- **Attachments**: Binary files referenced in document content
- **Hierarchy**: Parent-child document relationships
- **Templates**: Documents marked as templates appear in "New from template" picker
- **Lifecycle**: Draft -> Published -> Archived -> Deleted

### API Client

`OutlineClient` in `utils/outline_client.py` handles async REST API interactions:

**Operations** (all async):
- Documents: get, search, list, create, update, move, archive, unarchive, delete, restore
- Collections: list, get, get_documents, create, update, delete, export, export_all
- Comments: create, list, get
- Attachments: get_redirect_url, fetch_content
- AI: answer questions
- API Keys: list_api_keys (scope introspection for dynamic tool list)

**Connection Pooling**:
- Uses httpx with class-level connection pool
- Shared across all OutlineClient instances
- Automatic connection reuse for better performance
- Configurable limits via environment variables

**Rate Limiting**:
- Tracks `RateLimit-Remaining` and `RateLimit-Reset` headers, waits proactively when exhausted
- Uses asyncio.Lock for thread-safe rate limiting in concurrent scenarios
- Automatic retry with exponential backoff (max 3 attempts)
- Respects `Retry-After` header on HTTP 429 responses
- Enabled by default, no configuration required

**Error Handling**:
- Raises `OutlineError` for API failures
- Tools catch exceptions and return error strings
- Supports httpx exceptions (RequestError, HTTPStatusError, TimeoutException)

### Common Utilities (`features/documents/common.py`)

- `get_outline_client()` - Async function that creates an OutlineClient. Checks for a per-user Outline API key from the `x-outline-api-key` HTTP header first (SSE/streamable-http), then falls back to `OUTLINE_API_KEY` env var.
- `_get_header_api_key()` - Reads the `x-outline-api-key` header from the MCP SDK's `request_ctx` ContextVar. Returns `None` for stdio or when header is absent.
- `OutlineClientError` - Exception class for client-related errors

### Copilot CLI Patch (`patches/copilot_cli.py`)

Workaround for GitHub Copilot CLI sending `""` instead of `{}` for empty tool parameters.
Applied before server initialization. Patches `mcp.types.CallToolRequestParams` with a field validator.

## Implementation Patterns

### Module Structure


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Vortiago/mcp-outline](https://github.com/Vortiago/mcp-outline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
