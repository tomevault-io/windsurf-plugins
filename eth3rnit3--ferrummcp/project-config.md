---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FerrumMCP is a browser automation server implementing the Model Context Protocol (MCP). It provides AI assistants with browser automation capabilities through a standardized interface, using Ferrum (Ruby's headless Chrome driver) and optional BotBrowser integration for anti-detection.

## Architecture

### Core Components

**Server Layer** (`lib/ferrum_mcp/bin/ferrum-mcp`)
- `FerrumMCP::Server`: Main MCP server implementation
- Manages 27+ browser automation tools organized into 6 categories (Session Management, Navigation, Interaction, Extraction, Waiting, Advanced)
- Tools are defined in `TOOL_CLASSES` constant and registered with the MCP server at initialization
- **Session-based architecture**: All browser operations require an explicit session

**Session Management** (`lib/ferrum_mcp/session_manager.rb`, `lib/ferrum_mcp/session.rb`)
- `SessionManager`: Thread-safe session pool with automatic cleanup
- `Session`: Encapsulates a browser instance with custom configuration
- Supports multiple concurrent browser sessions with different configurations
- Each session can use different browser types (Chrome, BotBrowser), options, and profiles
- Session lifecycle: create → use → auto-cleanup (after 30min idle) or manual close
- **Important**: All browser tools require a valid `session_id` parameter

**Browser Management** (`lib/ferrum_mcp/browser_manager.rb`)
- `BrowserManager`: Handles Ferrum browser lifecycle
- Supports both standard Chrome/Chromium and BotBrowser (anti-detection mode)
- Browser options configured via `browser_options` method with anti-automation flags
- Each session has its own `BrowserManager` with custom configuration

**Transport Layer** (`lib/ferrum_mcp/transport/`)
- Two transport implementations:
  - `HTTPServer`: Uses MCP's `StreamableHTTPTransport` with Puma/Rack
  - `StdioServer`: Uses MCP's `StdioTransport` for standard I/O communication
- Transport is selected at startup via `--transport` flag (http or stdio)

**Tool Architecture** (`lib/ferrum_mcp/tools/`)
- All tools inherit from `BaseTool`
- Each tool must implement: `execute(params)`, `.tool_name`, `.description`, `.input_schema`
- Tools use `success_response`, `error_response`, or `image_response` helper methods
- `find_element` helper with timeout support for element location

**Configuration** (`lib/ferrum_mcp/configuration.rb`)
- Multi-browser and multi-profile support via structured ENV variables
- Supports multiple browsers, user profiles, and BotBrowser profiles
- File-only logging (no console output) to `logs/ferrum_mcp.log`
- Validates all configured browser paths at startup
- Backward compatible with legacy `BROWSER_PATH` and `BOTBROWSER_PROFILE` variables

**Resource Manager** (`lib/ferrum_mcp/resource_manager.rb`)
- Exposes server capabilities and configurations as MCP Resources
- Provides AI agents with discovery of available browsers and profiles
- Resources include: browsers, user profiles, bot profiles, and server capabilities
- All resources accessible via `ferrum://` URI scheme

### Dependency Management

Uses Zeitwerk for autoloading with custom inflections for acronyms (MCP, HTML, URL, JS). The loader is configured in `lib/ferrum_mcp.rb` and eager loads in production, lazy loads in development.

## Development Commands

### Running the Server

```bash
# Start with HTTP transport (default)
ruby bin/ferrum-mcp
# or explicitly
ruby bin/ferrum-mcp --transport http

# Start with STDIO transport (for MCP clients like Claude Desktop)
ruby bin/ferrum-mcp --transport stdio

# View help
ruby bin/ferrum-mcp --help

# View version
ruby bin/ferrum-mcp --version
```

### Testing

```bash
# Run all tests
bundle exec rspec

# Run specific test file
bundle exec rspec spec/ferrum_mcp/tools/navigation_tools_spec.rb

# Run tests with coverage
COVERAGE=true bundle exec rspec
```

Tests use a WEBrick test server on port 9999 started in `spec_helper.rb`. All tests run with headless browser and error-level logging.

### Linting

```bash
# Run RuboCop
bundle exec rubocop

# Auto-fix issues
bundle exec rubocop -A

# Via Rake
rake rubocop
rake rubocop_fix
```

### Rake Tasks

```bash
# Check environment configuration
rake check_env

# List all available tools
rake list_tools

# Run tests
rake test

# Generate a secure API key
rake generate_api_key
```

## Session Management

### Creating and Using Sessions

**All browser operations require a session**. You must create a session before using any browser automation tools:

```ruby
# 1. Create a session (returns session_id)
session_id = create_session(
  headless: true,
  timeout: 60,
  browser_options: { '--window-size': '1920,1080' }
)

# 2. Use the session_id with any browser tool
navigate(url: "https://example.com", session_id: session_id)
screenshot(session_id: session_id)

# 3. Close the session when done (or it auto-closes after 30min idle)
close_session(session_id: session_id)
```

### Multiple Concurrent Sessions

You can run multiple browsers in parallel with different configurations:

```ruby
# Standard Chrome for simple tasks
chrome_session = create_session(headless: true)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Eth3rnit3/FerrumMCP](https://github.com/Eth3rnit3/FerrumMCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
