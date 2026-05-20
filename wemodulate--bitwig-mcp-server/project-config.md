---
trigger: always_on
description: This project creates a server that allows Claude to control Bitwig Studio through the Model Context Protocol (MCP). It translates between MCP requests from Claude and OSC commands to Bitwig Studio's control API.
---

# Bitwig MCP Server - Claude Code Guide

## Project Overview

This project creates a server that allows Claude to control Bitwig Studio through the Model Context Protocol (MCP). It translates between MCP requests from Claude and OSC commands to Bitwig Studio's control API.

## Project-Specific Guidelines

### Development Workflow

For this project, follow these specific steps:

1. Use an incremental approach focusing on one feature at a time
2. Update/create appropriate tests for each change
3. Follow this test procedure for each change:
   - Run `pytest tests/` to verify changes
   - Run `mypy bitwig_mcp_server/` for type checking
   - Run `ruff check bitwig_mcp_server/` for linting
   - Run `black bitwig_mcp_server/` for code formatting

### MCP Implementation Guidelines

- Tools should follow the MCP Tool schema pattern
- Each tool should map to one or more OSC commands
- Error handling should be comprehensive and informative
- Consider user experience when designing tool APIs
- Resources should provide clear, structured information
- Use URIs that follow a consistent pattern
- Cache resource data appropriately to avoid excessive OSC traffic

## External References

### MCP SDK and Documentation

- MCP python-sdk GitHub Repository: https://github.com/modelcontextprotocol/python-sdk
- MCP documentation: https://modelcontextprotocol.io/
- GitHub Repository: https://github.com/anthropics/anthropic-sdk-python
- Documentation: https://docs.anthropic.com/

### Project-specific references

- coding guidelines: https://github.com/jxstanford/prompts-and-context/code
- MCP development guidelines: https://github.com/jxstanford/prompts-and-context/mcp
- Bitwig and OSC documentation: https://github.com/jxstanford/prompts-and-context/bitwig-mcp-server
- Bitwig OSC Reference: https://github.com/git-moss/DrivenByMoss-Documentation/blob/master/Generic-Tools-Protocols/Open-Sound-Control-(OSC).md

### Music Production Resources

Our prioritization of features is informed by standard music production workflows. For additional resources on music production workflows, consider the following common references:

- Bitwig Studio Manual: Official documentation covering Bitwig's workflow and features
- Ableton's "Making Music - 74 Creative Strategies": Excellent resource on music creation workflows
- Rick Snoman's "Dance Music Manual": Contains detailed production workflows
- "Mixing Secrets for the Small Studio" by Mike Senior: Covers workflow for the mixing phase
- "The Art of Music Production" by Richard James Burgess: Historical and practical music production processes

## Current Status

### Implemented Components

- OSC Communication Layer:
  - `BitwigOSCClient`: Sends OSC messages to Bitwig (`bitwig_mcp_server/osc/client.py`)
  - `BitwigOSCServer`: Receives OSC messages from Bitwig (`bitwig_mcp_server/osc/server.py`)
  - `BitwigOSCController`: Coordinates bidirectional communication (`bitwig_mcp_server/osc/controller.py`)
- MCP Server Framework:
  - `BitwigMCPServer`: Main server class integrating MCP and OSC (moved to `bitwig_mcp_server/mcp/server.py`)
  - `app.py`: High-level application coordinator (`bitwig_mcp_server/app.py`)
  - MCP Tools: Basic Bitwig control operations (`bitwig_mcp_server/mcp/tools.py`)
  - MCP Resources: Queryable Bitwig state information (`bitwig_mcp_server/mcp/resources.py`)
  - MCP Prompts: Templates for common workflows (`bitwig_mcp_server/mcp/prompts.py`)

### Development Workflow

When working on this codebase:

1. Use `pytest` for running tests: `pytest tests/`
2. For type checking: `mypy bitwig_mcp_server/`
3. For linting: `ruff check bitwig_mcp_server/`
4. For code formatting: `black bitwig_mcp_server/`

## Project Structure

### Core Components

- `bitwig_mcp_server/app.py`: Main application coordinator (high-level)
- `bitwig_mcp_server/settings.py`: Configuration settings
- `bitwig_mcp_server/mcp/`: Model Context Protocol implementation
  - `bitwig_mcp_server/mcp/server.py`: MCP server implementation
  - `bitwig_mcp_server/mcp/tools.py`: Tools Claude can use to control Bitwig
  - `bitwig_mcp_server/mcp/resources.py`: Resources Claude can query about Bitwig's state
  - `bitwig_mcp_server/mcp/prompts.py`: Templates for common music production workflows
- `bitwig_mcp_server/osc/`: Open Sound Control implementation
  - `bitwig_mcp_server/osc/client.py`: Client to send OSC messages to Bitwig
  - `bitwig_mcp_server/osc/server.py`: Server to receive OSC messages from Bitwig
  - `bitwig_mcp_server/osc/controller.py`: Coordinates OSC client and server
  - `bitwig_mcp_server/osc/error_handler.py`: Handles OSC communication errors
  - `bitwig_mcp_server/osc/exceptions.py`: Custom exceptions for OSC communication

### Testing Structure

- `tests/mcp/`: Unit tests for MCP components
  - `tests/mcp/test_server.py`: Tests for the MCP server
  - `tests/mcp/test_tools.py`: Tests for MCP tools
  - `tests/mcp/test_resources.py`: Tests for MCP resources
- `tests/osc/`: Unit tests for OSC components
- `tests/integration/`: Integration tests
  - `tests/integration/test_app_integration.py`: Tests for the application using MCP client

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WeModulate/bitwig-mcp-server](https://github.com/WeModulate/bitwig-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
