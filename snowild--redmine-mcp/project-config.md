---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an MCP (Model Context Protocol) server project for integrating with Redmine systems. Uses Python 3.12+ and the uv package manager.

## Development Environment Setup

### Dependency Management
- Use `uv` as the package manager
- Install dependencies: `uv sync`
- Run tests: `uv run python -m pytest`

### Project Structure
```
redmine-mcp/
├── src/redmine_mcp/          # Main source code
│   ├── __init__.py           # Package initialization
│   ├── server.py             # MCP server main program ✓ Completed
│   ├── redmine_client.py     # Redmine API client ✓ Completed
│   ├── config.py             # Configuration management ✓ Completed
├── tests/                    # Test files
├── docs/                     # Documentation directory
│   ├── issues/               # Development issue records
│   └── manuals/              # Technical manuals
├── pyproject.toml            # Project configuration and dependencies
├── uv.lock                   # Locked dependency versions
└── .env                      # Environment variables (to be created)
```

## MCP Development Notes

### Technology Stack
- **MCP SDK**: mcp[cli] >= 1.9.4 (uses FastMCP)
- **HTTP Client**: requests >= 2.31.0
- **Configuration Management**: python-dotenv >= 1.0.0
- **Image Processing**: Pillow >= 10.0.0
- **Python Version**: >= 3.12

### MCP Server Architecture
- Uses FastMCP to build the server
- Tool registration uses the `@mcp.tool()` decorator
- Supports asynchronous operations and type safety

### Redmine API Integration
- **Issue Management**: Query, create, update, delete issues
- **Project Management**: Query, create, update, delete, archive projects
- **User Management**: Query users, get current user information
- **Metadata Queries**: Status, priority, tracker lists
- **Watcher Management**: Add/remove issue watchers
- **Full Filter Support**: Multi-condition filtering and sorting

## Claude Code Integration

### Install to Claude Code
```bash
# Install MCP server
uv tool install .

# Or using pip
pip install .

# Add to Claude Code
claude mcp add redmine "redmine-mcp" \
  -e REDMINE_DOMAIN="https://your-redmine-domain.com" \
  -e REDMINE_API_KEY="your_api_key_here" \
  -e REDMINE_MCP_LOG_LEVEL="INFO" \
  -e REDMINE_MCP_TIMEOUT="30"
```

### Environment Variable Reference

To avoid conflicts with other projects' environment variables, redmine-mcp uses a dedicated prefix:

- **Required Variables**:
  - `REDMINE_DOMAIN`: Redmine server URL
  - `REDMINE_API_KEY`: Redmine API key

- **Log Level Control**:
  - `REDMINE_MCP_LOG_LEVEL`: Project-specific log level (default: INFO)
  - `FASTMCP_LOG_LEVEL`: Built-in FastMCP variable (optional)
    - If not set, the system will automatically use the value of `REDMINE_MCP_LOG_LEVEL`
    - Setting this variable allows separate control of FastMCP log output

- **Other Configuration**:
  - `REDMINE_MCP_TIMEOUT`: Request timeout (seconds)
  - `REDMINE_TIMEOUT`: Backward-compatible timeout setting

### Available MCP Tools (26)
- **Management Tools**: server_info, health_check, refresh_cache
- **Query Tools**: get_issue, list_project_issues, get_projects, get_issue_statuses, get_trackers, get_priorities, get_time_entry_activities, get_document_categories, search_issues, get_my_issues
- **Journal Tools**: list_issue_journals, get_journal
- **Attachment Tools**: get_attachment_info, get_attachment_image ✨ New (supports thumbnail and visual analysis)
- **User Tools**: search_users, list_users, get_user
- **Editing Tools**: update_issue_status, update_issue_content, add_issue_note (supports time entry recording), assign_issue, close_issue
- **Creation Tools**: create_new_issue (supports name parameters)

## Common Commands

```bash
# Install dependencies
uv sync

# Run MCP server
uv run python -m redmine_mcp.server

# Test Claude Code integration
uv run python tests/scripts/claude_integration.py

# Run unit tests
uv run python -m pytest tests/unit/

# Run all tests
uv run python -m pytest tests/

# Add new dependencies
uv add <package-name>
```

## Smart Cache System ✨

### Cache Mechanism Features
- **Multi-Domain Support**: Automatically creates independent cache files based on Redmine domain
- **Auto-Update**: Cache data automatically refreshes every 24 hours
- **Full Coverage**: Caches enum values (priority, status, tracker) and user data
- **File Location**: `~/.redmine_mcp/cache_{domain}_{hash}.json`

### Available Helper Functions
```python
client = get_client()

# Enum value queries
priority_id = client.find_priority_id_by_name("Low")           # → 5
status_id = client.find_status_id_by_name("In Progress")       # → 2
tracker_id = client.find_tracker_id_by_name("Bug")            # → 1

# User queries
user_id = client.find_user_id("Redmine Admin")              # Smart query (name or login)
user_id = client.find_user_id_by_name("Redmine Admin")      # Name-only query
user_id = client.find_user_id_by_login("admin")             # Login-only query

# Time entry activity queries
activity_id = client.find_time_entry_activity_id_by_name("Development")  # → 11

# Get all options

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [snowild/redmine-mcp](https://github.com/snowild/redmine-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
