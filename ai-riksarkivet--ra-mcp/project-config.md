---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**ra-mcp** (Riksarkivet Model Context Protocol) is an MCP server that provides access to transcribed historical documents from the Swedish National Archives (Riksarkivet).

### Architecture

The project is organized as a **uv workspace** with eight modular packages plus a root server:

```
ra-mcp/
├── src/ra_mcp_server/          # Root: Server composition and main CLI
│   ├── server.py               # FastMCP composition entry point
│   └── cli/app.py              # Typer CLI root (ra command)
├── packages/
│   ├── common/                 # ra-mcp-common: Shared HTTP client and utilities
│   ├── search/                 # ra-mcp-search: Search domain (models, clients, operations)
│   ├── browse/                 # ra-mcp-browse: Browse domain (models, clients, operations)
│   ├── search-mcp/             # ra-mcp-search-mcp: MCP tools for search
│   ├── browse-mcp/             # ra-mcp-browse-mcp: MCP tool for browse
│   ├── search-cli/             # ra-mcp-search-cli: CLI command for search
│   ├── browse-cli/             # ra-mcp-browse-cli: CLI command for browse
│   └── guide-mcp/              # ra-mcp-guide-mcp: MCP resources for historical guides
├── resources/                  # Historical guide markdown files
├── pyproject.toml              # Workspace configuration
└── uv.lock                     # Shared lockfile
```

### Package Structure

**ra-mcp-common** (no internal dependencies):
- [http_client.py](packages/common/src/ra_mcp_common/utils/http_client.py): Centralized urllib-based HTTP client with logging

**ra-mcp-search** (depends on common):
- [config.py](packages/search/src/ra_mcp_search/config.py): Search API URL and constants
- [models.py](packages/search/src/ra_mcp_search/models.py): Pydantic models (SearchRecord, RecordsResponse, SearchResult)
- [clients/search_client.py](packages/search/src/ra_mcp_search/clients/search_client.py): SearchAPI client
- [operations/search_operations.py](packages/search/src/ra_mcp_search/operations/search_operations.py): Search business logic

**ra-mcp-browse** (depends on common):
- [config.py](packages/browse/src/ra_mcp_browse/config.py): Browse API URLs and constants
- [models.py](packages/browse/src/ra_mcp_browse/models.py): Pydantic models (BrowseResult, PageContext)
- [clients/](packages/browse/src/ra_mcp_browse/clients/): API clients (ALTOClient, IIIFClient, OAIPMHClient)
- [operations/browse_operations.py](packages/browse/src/ra_mcp_browse/operations/browse_operations.py): Browse business logic
- [url_generator.py](packages/browse/src/ra_mcp_browse/url_generator.py): URL construction helpers

**ra-mcp-search-mcp** (depends on search + fastmcp):
- [tools.py](packages/search-mcp/src/ra_mcp_search_mcp/tools.py): FastMCP server setup, instructions, and tool registration
- [search_tool.py](packages/search-mcp/src/ra_mcp_search_mcp/search_tool.py): `search_transcribed` and `search_metadata` MCP tools
- [server.py](packages/search-mcp/src/ra_mcp_search_mcp/server.py): Standalone entry point for isolated dev/testing
- [formatter.py](packages/search-mcp/src/ra_mcp_search_mcp/formatter.py): Search result formatting for LLM output

**ra-mcp-browse-mcp** (depends on browse + fastmcp):
- [tools.py](packages/browse-mcp/src/ra_mcp_browse_mcp/tools.py): FastMCP server setup, instructions, and tool registration
- [browse_tool.py](packages/browse-mcp/src/ra_mcp_browse_mcp/browse_tool.py): `browse_document` MCP tool
- [server.py](packages/browse-mcp/src/ra_mcp_browse_mcp/server.py): Standalone entry point for isolated dev/testing
- [formatter.py](packages/browse-mcp/src/ra_mcp_browse_mcp/formatter.py): Browse result formatting for LLM output

**ra-mcp-search-cli** (depends on search + typer + rich):
- [app.py](packages/search-cli/src/search_cli/app.py): Typer sub-app
- [search_cmd.py](packages/search-cli/src/search_cli/search_cmd.py): `ra search` CLI command
- [formatter.py](packages/search-cli/src/ra_mcp_search_cli/formatter.py): CLI output formatting

**ra-mcp-browse-cli** (depends on browse + typer + rich):
- [app.py](packages/browse-cli/src/ra_mcp_browse_cli/app.py): Typer sub-app
- [browse_cmd.py](packages/browse-cli/src/ra_mcp_browse_cli/browse_cmd.py): `ra browse` CLI command
- [formatter.py](packages/browse-cli/src/ra_mcp_browse_cli/formatter.py): CLI output formatting

**ra-mcp-guide-mcp** (depends on common + fastmcp):
- [tools.py](packages/guide-mcp/src/ra_mcp_guide_mcp/tools.py): FastMCP server and MCP resources for historical guides from `resources/`
- [server.py](packages/guide-mcp/src/ra_mcp_guide_mcp/server.py): Standalone entry point for isolated dev/testing

**Root package — ra-mcp** (depends on all MCP and CLI packages):
- [server.py](src/ra_mcp_server/server.py): FastMCP composition server (imports search, browse, guide modules)
- [cli/app.py](src/ra_mcp_server/cli/app.py): Main Typer CLI entry point (`ra` command)

### Package Dependencies

```
ra-mcp-common              (no internal deps)
       ↑
ra-mcp-search              (depends on common)
ra-mcp-browse              (depends on common)
       ↑
ra-mcp-search-mcp          (depends on search + fastmcp)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AI-Riksarkivet/ra-mcp](https://github.com/AI-Riksarkivet/ra-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-06 -->
