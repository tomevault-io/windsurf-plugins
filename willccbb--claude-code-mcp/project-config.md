---
trigger: always_on
description: This project aims to leverage the Model Context Protocol (MCP) to enhance Claude's capabilities through external tools and APIs.
---

# Claude Code Helper Guide

## Project Architecture Overview

This project aims to leverage the Model Context Protocol (MCP) to enhance Claude's capabilities through external tools and APIs.

Key concepts:
- We use MCP servers from two sources:
  1. **Reference servers** - Located at `$HOME/dev/mcp-servers-repo` (cloned from [github.com/modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers))
  2. **Local servers** - Located at `$HOME/dev/claude-code/mcp-servers/` (our own implementations)

The reference MCP servers (from `MCP_REPO_PATH`) should be treated as read-only. Our local MCP servers (in `MCP_LOCAL_PATH`) will be our own implementations, some of which will be based on the reference servers but with extensions and improvements.

### Directory Structure
- `docs/` - Reference documentation
- `mcp-servers/` - Our local implementations of MCP servers
- `archive/` - Storage for deprecated or unused code components
- `data/` - Data storage for local servers (e.g., sqlite, memory)
- `scripts/` - Utility scripts for setup and testing
- `src/` - Source code for Python modules and utilities

## MCP Server Configuration

The primary configuration is handled by the `claude-mcp` script at the project root:

```bash
# Script path
/Users/williambrown/dev/claude-code/claude-mcp
```

Key environment variables:
- `CLAUDE_FILESYSTEM_PATH="$HOME/dev"` - Base path for filesystem access
- `MCP_REPO_PATH="$HOME/dev/mcp-servers-repo"` - Reference MCP servers (read-only)
- `MCP_LOCAL_PATH="$HOME/dev/claude-code/mcp-servers"` - Our local MCP server implementations
- `CLAUDE_MEMORY_PATH="$HOME/dev/claude-code/data/memory/memory.json"` - Memory file path
- `CLAUDE_SQLITE_PATH="$HOME/dev/claude-code/data/sqlite/test.db"` - SQLite database path

### MCP Server Registration
The script uses `claude mcp add` commands to register servers with Claude. It uses two types of servers:

1. **Package-based servers** (from npm/pypi):
   ```bash
   # Examples
   claude mcp add filesystem -- npx -y @modelcontextprotocol/server-filesystem $CLAUDE_FILESYSTEM_PATH
   claude mcp add fetch -- uvx mcp-server-fetch
   ```

2. **Local servers** (from our repository):
   ```bash
   # Example
   claude mcp add sqlite -- uv --directory "${MCP_REPO_PATH}/src/sqlite" run mcp-server-sqlite --db-path $CLAUDE_SQLITE_PATH
   ```

### Required Environment Variables
- `BRAVE_API_KEY` - For brave-search server
- `GITHUB_TOKEN` - For GitHub operations (formerly GITHUB_PERSONAL_ACCESS_TOKEN)
- `LINEAR_API_KEY` - For Linear issue tracking
- `SLACK_BOT_TOKEN` and `SLACK_TEAM_ID` - For Slack communication
- `E2B_API_KEY` - For code execution sandbox
- `ALLOWED_PATHS` - For filesystem access control (defaults to project root)

### Important MCP Servers
- **brave-search** - Web search capabilities
- **github** - GitHub repository operations
- **filesystem** - Local file system operations
- **fetch** - Web page fetching
- **memory** - Knowledge graph for persistent memory
- **sqlite** - Database operations
- **slack** - Slack communication
- **linear** - Linear issue tracking integration
- **e2b** - Code execution sandbox
- **research-papers** - Research paper management with Semantic Scholar integration
- **mcp-test** - Testing client for developing and testing other MCP servers

### MCP Server Usage Guidelines

When to use specific MCP servers:

- **brave-search**: Use for general web searches and finding current information.
  - Best for: Finding documentation, articles, tutorials, product information
  - Example: `mcp__brave-search__brave_web_search(query="python type hints tutorial")`

- **github**: Use for interacting with GitHub repositories.
  - Best for: Examining code, creating issues/PRs, searching repos, accessing GitHub content
  - Example: `mcp__github__search_repositories(query="semantic scholar api")`

- **filesystem**: Use for reading and manipulating local files.
  - Best for: Reading, writing, and exploring files in allowed directories
  - Example: `mcp__filesystem__read_file(path="/path/to/file")`

- **fetch**: Use for downloading web content directly.
  - Best for: Reading web articles, documentation, accessing external APIs
  - Example: `mcp__fetch__fetch(url="https://example.com/api/docs")`

- **docker**: Use for running isolated code and managing development environments.
  - Best for: Executing code snippets, running code with dependencies, creating custom environments
  - Run code in ephemeral containers with various languages (with internet access)
  - Register custom Docker templates via tool calls
  - Support for Python, Node.js, Ruby, and more
  - Example: `mcp__docker__docker_run_code(language="python", code="import requests\nprint(requests.get('https://httpbin.org/get').json())", dependencies=["requests"])`
  - Example: `mcp__docker__docker_register_template(name="custom-py", image="python:3.11-slim", description="Custom Python environment")`

- **memory**: Use for persistent knowledge storage across sessions.
  - Best for: Storing user preferences, frequently used commands, project context
  - Example: `mcp__memory__create_entities(...)`

- **sqlite**: Use for structured data storage and querying.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [willccbb/claude-code-mcp](https://github.com/willccbb/claude-code-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
