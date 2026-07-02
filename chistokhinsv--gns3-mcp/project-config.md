---
trigger: always_on
description: Project-specific instructions for working with the GNS3 MCP server codebase.
---

# GNS3 MCP Server - Project Instructions

Project-specific instructions for working with the GNS3 MCP server codebase.

## Project Overview

MCP server providing programmatic access to GNS3 network simulation labs. Includes:
- Desktop extension (.mcpb) for Claude Desktop
- Agent skill with GNS3 procedural knowledge
- Console management for device interaction
- GNS3 v3 API client with JWT authentication

## Current Version: v0.54.0 (MCP Server) / v0.3.0 (SSH Proxy)

**Latest Release:** v0.54.0 - GNS3 Notification Stream Support
**SSH Proxy:** v0.3.0 - Multi-Service Architecture (TFTP server, HTTP reverse proxy, HTTP client)

### Recent Changes (v0.26.0 - v0.54.0)

**v0.54.0** - GNS3 notification stream support
- **New `notification` tool**: Subscribe to `/v3/notifications` real-time event stream
- Actions: subscribe (controller/project-level), read (diff/all/last + filter), unsubscribe, status
- Event types: node/link/drawing CRUD, log.error/warning/info, compute.*, template.*, ping
- Background stream reader with auto-reconnect, buffer management (5000 max)
- **Tool count**: 13 tools (up from 12)

**v0.48.0** - List action integration in CRUD tools
- **Removed**: `list_projects()`, `list_nodes()`, `get_topology()` convenience wrappers (3 tools removed)
- **Added**: `list` action to `project()`, `node()`, `link()`, `drawing()` CRUD tools
- **Consistent Pattern**: All CRUD tools now have `list` action with `project_id` and `format` parameters
- **Tool Count**: 12 tools (down from 15 in v0.47.0) - 20% reduction
- **search_tools()**: Updated TOOL_REGISTRY with new actions

**SSH Proxy v0.3.0** - Multi-Service Architecture (TFTP, HTTP reverse proxy, HTTP client)
- **TFTP Server**: tftpd-hpa on port 69/udp, read-write access, `/opt/gns3-ssh-proxy/tftp` root
- **HTTP Reverse Proxy**: nginx on port 8023, URL: `http://proxy:8023/http-proxy/<ip>:<port>/`
- **HTTP Client**: RESTful endpoint for making HTTP/HTTPS requests to device APIs
- **MCP Tools**: Added `tftp` and `http_client` CRUD-style tools (14 tools total now)
- **Supervisor**: Runs FastAPI, TFTP, and nginx in single container
- **GitHub Actions**: Manual workflow for Docker build/push with automated Docker Hub description updates
- **Documentation**: Updated ssh-proxy/README.md with features, architecture, API endpoints, usage examples

**v0.47.0** - **BREAKING**: Aggressive tool consolidation (32 → 15 tools, 53% reduction)
- **Core CRUD Tools**: 7 consolidated tools with `action` parameters
  - `gns3_connection()`, `project()`, `node()`, `link()`, `drawing()`, `project_docs()`, `snapshot()`
- **Batch-Only Console/SSH**: Removed 8 individual tools, batch operations only
  - `console(operations=[...])` and `ssh(operations=[...])` with two-phase validation
- **Tool Discovery**: `search_tools()` with category/capability/resource filtering
- **Wildcard Bulk Operations**: Node operations support `"*"`, `"Router*"`, JSON arrays with parallel execution
- **No Backward Compatibility**: Breaking changes, all individual tools replaced
- **Prompts Updated**: All 5 workflow prompts use new CRUD-style APIs
- **Tests Pass**: Fixed `Literal` import error, all 202 tests pass
**v0.46.0** - Resource query tools for Claude Desktop compatibility (DEPRECATED in v0.48.0)
- **4 New Tools**: `query_resource()`, `list_projects()`, `list_nodes()`, `get_topology()`
- **Claude Desktop Support**: All 25+ resources now accessible via tools
- **Convenience Shortcuts**: 3 high-use operations (projects, nodes, topology) - removed in v0.48.0
- **Universal Query**: `query_resource()` remains for advanced use cases
- **Note**: v0.48.0 integrated list functionality into CRUD tools, removing the 3 convenience wrappers
**v0.40.0** - Wildcard bulk operations, topology report resource, structured exceptions
- **Bulk Node Operations**: Wildcard patterns (`"*"`, `"Router*"`), JSON arrays, parallel execution
- **Topology Report**: Single-resource overview of nodes/links/statistics (replaces 3+ tool calls)
- **Structured Exceptions**: GNS3Error hierarchy with error codes and suggestions
- **Performance**: 5-10× faster bulk operations, backward compatible
- **Enhanced set_node_properties**: Supports `"*"` for all nodes, `"Router*"` for pattern matching
**v0.39.0** - Phase 1: Server instructions (170-line AI guidance) + progress notifications (node start + SSH)
- Server instructions loaded automatically, guide AI on GNS3-specific behaviors
- Progress notifications for node start (12 steps, 5s polling) and SSH commands (wait_timeout > 10s)
- Enhanced parameter descriptions with validation hints (alternative to completions)
- Note: Argument completions not available (FastMCP Python limitation, TypeScript only)
**v0.30.0** - **BREAKING**: Table mode output, URIs instead of IDs, proxy type field
**v0.29.1** - Dual access patterns for sessions (path-based + query-param)
**v0.29.0** - **BREAKING**: Resource URI scheme changes, complete metadata, linting infrastructure
**v0.28.0** - Local execution on SSH proxy container (ssh_command with node_name="@")
**v0.27.0** - Configurable SSH session timeouts
**v0.26.0** - Multi-proxy support for isolated network access

### Current State
- **15 Tools**: CRUD-style consolidation (v0.54.0: added notification tool)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChistokhinSV/gns3-mcp](https://github.com/ChistokhinSV/gns3-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
