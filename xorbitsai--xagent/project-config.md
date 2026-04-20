---
trigger: always_on
description: Xagent is a powerful and flexible framework for building and running AI-powered agents with support for various execution patterns, tools, memory management, and observability.
---

# Xagent Agent System

Xagent is a powerful and flexible framework for building and running AI-powered agents with support for various execution patterns, tools, memory management, and observability.

## Features

- **Agent Patterns**: ReAct, DAG plan-execute
- **Nested Agents**: Hierarchical agent execution with parent-child relationships
- **Tool System**: Built-in tools with auto-discovery mechanism
- **Memory Management**: LanceDB-based vector storage with semantic search
- **Observability**: Langfuse integration for tracing and monitoring
- **Real-time Communication**: WebSocket support for agent execution monitoring

## Architecture Overview

### Entry Points

Xagent has one main entrypoint:

**Web Interface (`src/xagent/web/`):**
- FastAPI-based web application with WebSocket support
- Real-time agent execution monitoring
- File upload and management
- DAG visualization
- API endpoints for agent operations

## Architecture Overview

### Core Components (`src/xagent/core/`)

**Agent System:**
- `agent.py` - Main Agent class with nested agent support and execution history
- `pattern/` - Agent execution patterns (ReAct, DAG plan-execute)
- `runner.py` - Agent execution engine
- `context.py` - Agent context management

**Graph System:**
- `graph.py` - Graph workflow execution engine with validation
- `node.py` - Node types (Start, End, Agent, Tool, etc.)
- `node_factory.py` - Node creation factory

**Tools System:**
- `adapters/` - Tool adapters for different frameworks
- `core/` - Core tool implementations (calculator, file operations, web search, etc.)
- Tool auto-discovery using `get_{tool_name}_tool()` naming convention

**Model Integration:**
- `llm/` - LLM provider implementations (OpenAI, Zhipu)
- Support for embedding models and reranking models

**Memory Management:**
- `storage/` - Storage manager and database operations
- `workspace.py` - Task workspace management with isolated working directories

**Observability:**
- Langfuse integration for tracing and monitoring
- Execution history and message tracking

### Configuration System

**IMPORTANT**: All path-related and configuration settings SHALL try their best to use the unified configuration module at `src/xagent/config.py`.

**Core Principles:**
1. **Single source of truth** - All configuration goes through `config.py`
2. **No hardcoded paths** - Never use string literals like `"uploads"` or `"./data"`
3. **Environment variable support** - All paths must be configurable via `XAGENT_*` env vars
4. **No circular dependencies** - `config.py` has no dependencies on other core submodules

**Import Style:**
- **Source code** (`src/xagent/`): Use relative imports
  ```python
  from ..config import get_uploads_dir, get_storage_root
  ```
- **Test files** (`tests/`): Use absolute imports
  ```python
  from xagent.config import get_uploads_dir, get_storage_root
  ```

**Configuration Pattern:**
```python
def get_<config_name>() -> ReturnType:
    """Get <config> with environment variable override.

    Priority:
        1. <ENV_VAR> environment variable
        2. Computed default

    Returns:
        Description of return value
    """
    env_value = os.getenv(ENV_VAR)
    if env_value:
        return <process_env_value>(env_value)

    # Default computation
    return <compute_default>()
```

**Adding New Configuration:**
1. Add function to `src/xagent/config.py`
2. Add env var constant: `<ENV_VAR> = "XAGENT_<NAME>"`
3. Follow env var → default priority pattern
4. Update `example.env` with documentation
5. Add tests to `tests/core/test_config.py`

### Available Tools

Xagent has two categories of tools:

**Basic Tools** (`src/xagent/core/tools/core/`):
- `calculator` - Mathematical expression evaluation
- `file_tool` - File operations (read, write, list, edit, delete)
- `workspace_file_tool` - Workspace file operations
- `python_executor` - Dynamic Python code execution
- `browser_use` - Browser automation
- `excel` - Excel file operations
- `document_parser` - Document parsing (PDF, DOCX, etc.)
- `image_tool` - Image processing

**Web & Search Tools** (`src/xagent/core/tools/core/`):
- `web_search` - Generic web search
- `image_web_search` - Image search functionality
- `zhipu_web_search` - Zhipu search integration
- `web_crawler` - Web crawling and content extraction

**RAG Tools** (`src/xagent/core/tools/core/RAG_tools/`):
- Document parsing and chunking
- Vector storage and retrieval (LanceDB)
- Knowledge base management
- Semantic search capabilities

**MCP Server Tools** (`src/xagent/core/tools/core/mcp/`):
- Model Context Protocol (MCP) server integration
- Standardized tool access via MCP protocol

**Skill Documentation Access Tools** (`src/xagent/core/tools/adapters/vibe/skill_tools.py`):
- `read_skill_doc` - Read documentation from skill directories (SKILL.md, examples, etc.)
- `list_skill_docs` - List documentation files in skill directories (returns names and sizes)
- `fetch_skill_file` - Copy resource files from skill directories to workspace

### Custom Tools

Create custom tools by adding Python files following the naming convention:

```python
from langchain_core.tools import BaseTool, tool


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xorbitsai/xagent](https://github.com/xorbitsai/xagent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
