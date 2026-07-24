---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development
- Install core dependencies: `pip install -e .`
- Install dev dependencies: `pip install -e ".[dev]"`
- Install specific feature groups:
  - Document chat features: `pip install -e ".[doc-chat]"`
  - Database features: `pip install -e ".[db]"`
  - HuggingFace embeddings: `pip install -e ".[hf-embeddings]"`
  - All features: `pip install -e ".[all]"`
- Run linting and type checking: `make check`
- Format code: `make lint`

### Testing
- Run all tests: `pytest tests/`
- Run specific test: `pytest tests/main/test_file.py::test_function`
- Run tests with coverage: `pytest --cov=langroid tests/`
- Run only main tests: `make tests` (uses `pytest tests/main`)

### Linting and Type Checking
- Lint code: `make check` (runs black, ruff check, mypy)
- Format only: `make lint` (runs black and ruff fix)
- Type check only: `make type-check`
- Always use `make check` to run lints + mypy before trying to commit changes

### Version and Release Management
- Bump version: `./bump_version.sh [patch|minor|major]`
- Or use make commands:
  - `make all-patch` - Bump patch version, build, push, release
  - `make all-minor` - Bump minor version, build, push, release
  - `make all-major` - Bump major version, build, push, release

## Architecture

Langroid is a framework for building LLM-powered agents that can use tools and collaborate with each other.

### Core Components:

1. **Agents** (`langroid/agent/`):
   - `chat_agent.py` - Base ChatAgent that can converse and use tools
   - `task.py` - Handles execution flow for agents
   - `special/` - Domain-specific agents (doc chat, table chat, SQL chat, etc.)
   - `openai_assistant.py` - Integration with OpenAI Assistant API

2. **Tools** (`langroid/agent/tools/`):
   - Tool system for agents to interact with external systems
   - `tool_message.py` - Protocol for tool messages
   - Various search tools (Google, DuckDuckGo, Tavily, Exa, etc.)

3. **Language Models** (`langroid/language_models/`):
   - Abstract interfaces for different LLM providers
   - Implementations for OpenAI, Azure, local models, etc.
   - Support for hundreds of LLMs via LiteLLM

4. **Vector Stores** (`langroid/vector_store/`):
   - Abstract interface and implementations for different vector databases
   - Includes support for Qdrant, Chroma, LanceDB, Pinecone, PGVector, Weaviate

5. **Document Processing** (`langroid/parsing/`):
   - Parse and process documents from various formats
   - Chunk text for embedding and retrieval
   - Support for PDF, DOCX, images, and more

6. **Embedding Models** (`langroid/embedding_models/`):
   - Abstract interface for embedding generation
   - Support for OpenAI, HuggingFace, and custom embeddings

### Key Multi-Agent Patterns:

- **Task Delegation**: Agents can delegate tasks to other agents through hierarchical task structures
- **Message Passing**: Agents communicate by transforming and passing messages
- **Collaboration**: Multiple agents can work together on complex tasks

### Key Security Features:

- The `full_eval` flag in both `TableChatAgentConfig` and `VectorStoreConfig` controls code injection protection
- Defaults to `False` for security, set to `True` only in trusted environments

## Documentation

- Main documentation is in the `docs/` directory
- Examples in the `examples/` directory demonstrate usage patterns
- Quick start examples available in `examples/quick-start/`

## MCP (Model Context Protocol) Tools Integration

Langroid provides comprehensive support for MCP tools through the `langroid.agent.tools.mcp` module. Here are the key patterns and approaches:

### MCP Tool Creation Methods

#### 1. Using the `@mcp_tool` Decorator (Module Level)
```python
from langroid.agent.tools.mcp import mcp_tool
from fastmcp.client.transports import StdioTransport

transport = StdioTransport(command="...", args=[...])

@mcp_tool(transport, "tool_name")
class MyTool(lr.ToolMessage):
    async def handle_async(self):
        result = await self.call_tool_async()
        # custom processing
        return result
```

**Important**: The decorator creates the transport connection at module import time, so it must be used at module level (not inside async functions).

#### 2. Using `get_tool_async` (Inside Async Functions)
```python
from langroid.agent.tools.mcp.fastmcp_client import get_tool_async

async def main():
    transport = StdioTransport(command="...", args=[...])
    BaseTool = await get_tool_async(transport, "tool_name")
    
    class MyTool(BaseTool):
        async def handle_async(self):
            result = await self.call_tool_async()
            # custom processing
            return result
```

**Use this approach when**:
- Creating tools inside async functions
- Need to avoid event loop conflicts
- Want to delay transport creation until runtime

### Transport Types and Event Loop Considerations

- **StdioTransport**: Creates subprocess immediately, can cause "event loop closed" errors if created at module level in certain contexts
- **SSETransport**: HTTP-based, generally safer for module-level creation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [langroid/langroid](https://github.com/langroid/langroid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
