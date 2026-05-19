---
trigger: always_on
description: `mcp-client-cli` is a CLI tool that acts as a [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) client. It allows users to run LLM prompts from the terminal with access to MCP-compatible servers/tools. The CLI supports multiple LLM providers (OpenAI, Anthropic, Google) and can be extended with any MCP-compatible server.
---

# AGENTS.md

## Project Overview

`mcp-client-cli` is a CLI tool that acts as a [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) client. It allows users to run LLM prompts from the terminal with access to MCP-compatible servers/tools. The CLI supports multiple LLM providers (OpenAI, Anthropic, Google) and can be extended with any MCP-compatible server.

**Key Features:**
- MCP client implementation for tool usage
- Multi-provider LLM support (OpenAI, Anthropic, Google, local models)
- Conversation persistence and continuation
- Prompt templates
- Image input support (via stdin or clipboard)
- Memory storage for context across sessions

## Architecture

The project uses:
- **LangChain/LangGraph**: For LLM orchestration and agent execution
- **MCP SDK**: For communicating with MCP servers
- **SQLite**: For conversation checkpointing and memory storage
- **Rich**: For terminal output formatting

### Core Flow

1. User provides a query via CLI
2. `cli.py` parses arguments and loads configuration
3. MCP servers are initialized and tools are loaded
4. A LangGraph ReAct agent processes the query with available tools
5. Results are streamed back to the user with Rich formatting

## Directory Structure

```
mcp-client-cli/
├── src/mcp_client_cli/
│   ├── cli.py       # Main entry point, argument parsing, agent orchestration
│   ├── config.py    # Configuration loading and dataclasses
│   ├── const.py     # Constants (paths, defaults)
│   ├── input.py     # Input handling (clipboard support)
│   ├── memory.py    # SQLite-based memory/store implementation
│   ├── output.py    # Output formatting with Rich
│   ├── prompt.py    # Prompt templates
│   ├── storage.py   # Tool caching and conversation management
│   └── tool.py      # MCP to LangChain tool conversion
├── .github/workflows/
│   ├── test.yml     # Installation tests on Linux/Windows/macOS
│   └── release.yml  # PyPI publishing workflow
├── pyproject.toml   # Project metadata and dependencies
├── CONFIG.md        # Configuration documentation
└── README.md        # User-facing documentation
```

## Development Setup

### Prerequisites

- Python 3.12+
- [uv](https://github.com/astral-sh/uv) (recommended) or pip

### Installation

```bash
# Clone the repository
git clone https://github.com/adhikasp/mcp-client-cli.git
cd mcp-client-cli

# Create virtual environment and install in editable mode
uv venv
source .venv/bin/activate  # Linux/macOS
# .venv\Scripts\activate   # Windows

uv pip install -e .
```

### Configuration

Create `~/.llm/config.json` with your LLM and MCP server settings. See `CONFIG.md` for the full schema.

## Code Style and Conventions

### General Guidelines

- Use Python type hints throughout
- Follow PEP 8 style guidelines
- Use dataclasses for configuration objects
- Prefer async/await for I/O operations
- Use `commentjson` for config parsing (allows `//` comments in JSON)

### Key Patterns

1. **Configuration**: Use dataclasses in `config.py` with `from_dict` class methods
2. **Tools**: MCP tools are wrapped as LangChain `BaseTool` subclasses
3. **State Management**: Use `TypedDict` for agent state (see `AgentState` in `cli.py`)
4. **Output**: Use Rich library for formatted terminal output

### Error Handling

- Tool errors should raise `ToolException` for graceful handling
- Configuration errors should raise `FileNotFoundError` with helpful messages
- Network/async errors should include timeouts to prevent hanging

## Key Components

### `cli.py`
Main entry point. Contains:
- `run()`: Async main function
- `setup_argument_parser()`: CLI argument definitions
- `handle_conversation()`: Agent execution flow
- `parse_query()`: Input parsing (text, images, clipboard)

### `tool.py`
MCP-LangChain bridge:
- `McpToolkit`: Manages MCP server connection and tools
- `McpTool`: LangChain tool wrapper for MCP tools
- `convert_mcp_to_langchain_tools()`: Main conversion function

### `config.py`
Configuration management:
- `AppConfig`: Main config class with `load()` method
- `LLMConfig`: LLM provider settings
- `ServerConfig`: MCP server settings

### `memory.py`
Persistent storage:
- `SqliteStore`: LangGraph-compatible store implementation
- `save_memory`: Tool for LLM to save user memories
- `get_memories`: Retrieve stored memories

## Testing

Currently, the project has installation verification tests that run on all platforms (Linux, Windows, macOS). These tests verify:

1. Package installs correctly
2. CLI entry point works (`llm --help`)

To run locally:

```bash
llm --help
```

## CI/CD

### Test Workflow (`.github/workflows/test.yml`)
- Triggers on push/PR to main/master
- Tests installation on Ubuntu, Windows, macOS
- Uses Python 3.12

### Release Workflow (`.github/workflows/release.yml`)
- Triggers on push to master when commit message contains "release"
- Builds and publishes to PyPI
- Creates GitHub release with auto-generated notes

## Common Tasks

### Adding a New CLI Flag

1. Add argument in `setup_argument_parser()` in `cli.py`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adhikasp/mcp-client-cli](https://github.com/adhikasp/mcp-client-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
