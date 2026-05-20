---
trigger: always_on
description: This document outlines the essential rules and conventions for the PolarionMcpServers project. Follow these guidelines to maintain consistency and ensure proper functionality.
---

# PolarionMcpServers Developer Guidelines

This document outlines the essential rules and conventions for the PolarionMcpServers project. Follow these guidelines to maintain consistency and ensure proper functionality.

## Project Structure

- **PolarionMcpTools**: Core library with tools for interacting with Polarion
- **PolarionMcpServer**: Console application with stdio transport
- **PolarionRemoteMcpServer**: Web application with HTTP transport

## Build and Test

Use the Python build automation script for all build and run operations:

```bash
python build.py build    # Build solution (auto-stops running app)
python build.py start    # Build and start in background (port 5090)
python build.py stop     # Stop background application
python build.py status   # Check if application is running
python build.py run      # Run in foreground (blocks terminal)
```

### MCP Commands (Model Context Protocol)
```bash
python build.py mcp ping [--project <alias>]                    # Check MCP server connectivity
python build.py mcp info [--project <alias>]                    # Show MCP server information
python build.py mcp tools [--project <alias>]                   # List available MCP tools
python build.py mcp call <tool> '{"arg": "value"}' [--project <alias>]  # Call an MCP tool with JSON args
```

**MCP Examples:**
```bash
# Default project
python build.py mcp call search_workitems '{"searchQuery": "timeout", "maxResults": 10}'

# Specific project
python build.py mcp call search_workitems '{"searchQuery": "requirement", "itemTypes": "requirement"}' --project myproject-ext
python build.py mcp tools --project myproject-ext

# Query current work items in module
python build.py mcp call get_workitems_in_module '{"space": "MySpace", "documentId": "my_requirements_doc"}'

# Query work items at specific document revision
python build.py mcp call get_workitems_in_module '{"space": "MySpace", "documentId": "my_requirements_doc", "revision": "12345"}'

# With type filtering (current revision only)
python build.py mcp call get_workitems_in_module '{"space": "MySpace", "documentId": "my_requirements_doc", "itemTypes": "requirement"}'
```

**Available project aliases:** Configured in `appsettings.json` PolarionProjects array, overridden by `appsettings.Development.json` when running locally (default from appsettings or POLARION_DEFAULT_PROJECT env var)

**Note:** Use double quotes for JSON argument keys/values on Windows. If tools error with authentication failures, check that credentials are configured in `PolarionRemoteMcpServer/appsettings.Development.json` (or `appsettings.json` for production).

### REST API Commands

The REST API is designed to align with the **official Polarion REST API** specification found at `https://testdrive.polarion.com/polarion/rest/v1/definition`. A local copy of this definition is maintained at `docs/polarion-rest-vq-definition.json` for reference when implementing or extending endpoints.

```bash
python build.py rest <method> <path> [options]
```

**REST Options:**

- `--project <alias>` - Project to use (default: from appsettings or POLARION_DEFAULT_PROJECT env var). Use `{project}` placeholder in path
- `--query <text>` - Search query
- `--types <types>` - Comma-separated work item types
- `--status <status>` - Comma-separated status values
- `--sort <field>` - Sort field (can prefix with `-` for descending)
- `--page-size <n>` - Results per page (for `page[size]` parameter)
- `--limit <n>` - Limit results
- `--revision <n>` - Document revision number (for document workitem queries)
- `--format <fmt>` - Output format: `pretty` (default) or `raw`

**REST Examples:**

```bash
# Health check
python build.py rest GET api/health

# Search work items (new endpoint)
python build.py rest GET "polarion/rest/v1/projects/{project}/workitems" --query timeout --project myproject
python build.py rest GET "polarion/rest/v1/projects/{project}/workitems" --query requirement --types requirement,testCase --page-size 10 --project myproject-ext

# Get specific work item
python build.py rest GET "polarion/rest/v1/projects/{project}/workitems/WI-12345" --project myproject

# Get work item revisions
python build.py rest GET "polarion/rest/v1/projects/{project}/workitems/WI-12345/revisions" --limit 5 --project myproject

# Get outgoing linked work items
python build.py rest GET "polarion/rest/v1/projects/{project}/workitems/WI-12345/linkedworkitems" --project myproject

# Get incoming (back-linked) work items
python build.py rest GET "polarion/rest/v1/projects/{project}/workitems/WI-12345/backlinkedworkitems" --project myproject

# List spaces
python build.py rest GET "polarion/rest/v1/projects/{project}/spaces" --project myproject

# List documents in space
python build.py rest GET "polarion/rest/v1/projects/{project}/spaces/MySpace/documents" --project myproject

# Get work items in document (current revision)
python build.py rest GET "polarion/rest/v1/projects/{project}/spaces/MySpace/documents/MyDocument/workitems" --project myproject

# Get work items in document at specific revision

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [peakflames/PolarionMcpServers](https://github.com/peakflames/PolarionMcpServers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
