---
trigger: always_on
description: The `llama-index-server` package is a FastAPI-based server framework for deploying LlamaIndex Workflows and Agent Workflows as a high-performance API server with an optional chat UI. It provides a complete environment for running LlamaIndex workflows with both API endpoints and a user interface for interaction.
---

# LlamaIndex Server (Python)

## Overview

The `llama-index-server` package is a FastAPI-based server framework for deploying LlamaIndex Workflows and Agent Workflows as a high-performance API server with an optional chat UI. It provides a complete environment for running LlamaIndex workflows with both API endpoints and a user interface for interaction.

## Package Structure

### Core Components
- **`llama_index/server/server.py`**: Main `LlamaIndexServer` class extending FastAPI
- **`llama_index/server/__init__.py`**: Package exports (`LlamaIndexServer`, `UIConfig`, `UIEvent`)
- **`pyproject.toml`**: Package configuration with dependencies and build settings

### Key Directories
- **`api/`**: FastAPI routers, models, and request handling
- **`services/`**: Business logic for file handling, LlamaCloud integration, and UI generation
- **`tools/`**: Document generation, interpreter tools, and index querying utilities
- **`gen_ui/`**: AI-powered UI component generation system
- **`resources/`**: Static assets and bundled UI files
- **`examples/`**: Sample workflows demonstrating different features

## Core Functionality

### LlamaIndexServer Class
Main server implementation that extends FastAPI with workflow-specific features:
- **Workflow Factory Pattern**: Creates workflow instances per request using factory functions
- **UI Configuration**: Manages chat interface, custom components, and layout directories
- **File Serving**: Automatically mounts `data/` and `output/` directories
- **Development Mode**: Enables CORS, verbose logging, and hot reloading

### Chat API (`api/routers/chat.py`)
- **Endpoint**: `/api/chat` for chat interactions
- **Streaming Responses**: Real-time workflow execution with Vercel-compatible streaming
- **Message Handling**: Converts between API and LlamaIndex message formats
- **Background Tasks**: File downloads and asynchronous processing
- **LlamaCloud Integration**: Optional index selector for cloud-based retrieval

### Event System (`api/models.py`)
Structured event types for workflow communication:
- **`UIEvent`**: Custom UI component rendering with Pydantic data models
- **`ArtifactEvent`**: Code and document artifacts for Canvas panel display
- **`SourceNodesEvent`**: Document sources with metadata and file URLs
- **`AgentRunEvent`**: Agent tool usage and progress tracking

### UI Generation (`gen_ui/main.py`)
AI-powered component generation using LLM workflows:
- **`GenUIWorkflow`**: Multi-step process for creating React components
- **Planning Phase**: Analyzes event schemas to design UI layouts
- **Aggregation Logic**: Groups events for optimized rendering
- **Code Generation**: Creates shadcn/ui components with proper imports
- **Validation**: Ensures generated code uses only supported dependencies

## Development Environment

### Dependencies
```toml
# Core FastAPI server with standard extensions
fastapi[standard]>=0.115.11,<1.0.0

# LlamaIndex core and workflow engine
llama-index-core>=0.12.28,<1.0.0

# File handling and cloud integration
llama-index-readers-file>=0.4.6,<1.0.0
llama-index-indices-managed-llama-cloud>=0.6.3,<1.0.0

# HTTP requests and caching
requests>=2.32.3,<3.0.0
cachetools>=5.5.2,<6.0.0
pydantic-settings>=2.8.1,<3.0.0
```

### Development Dependencies
- **Testing**: pytest, pytest-asyncio, pytest-mock for comprehensive testing
- **Code Quality**: black, ruff, mypy, pylint for code formatting and linting
- **Documentation**: jupyter, markdown for examples and documentation
- **Integrations**: e2b-code-interpreter, llama-cloud for extended functionality

### Build System
- **Backend**: Hatchling for Python package building
- **Artifacts**: Includes `llama_index/server/resources` for bundled UI assets
- **Type Checking**: MyPy with strict settings for type safety

## Configuration Options

### Server Configuration
```python
LlamaIndexServer(
    workflow_factory=create_workflow,  # Required: factory function
    env="dev",                        # Environment: "dev" enables CORS and UI
    ui_config={                       # Optional UI configuration
        "enabled": True,              # Enable chat interface
        "starter_questions": [...],   # Predefined user prompts
        "component_dir": "components", # Custom UI components directory
        "layout_dir": "layout",       # Custom layout sections directory
        "dev_mode": True,             # Enable live code editing
        "llamacloud_index_selector": False, # LlamaCloud integration
    },
    suggest_next_questions=True,      # Auto-generate follow-up questions
    verbose=True,                     # Enable detailed logging
    api_prefix="/api",               # API route prefix
    server_url="http://localhost:8000", # Deployment URL
)
```

### Workflow Factory Contract
```python
def create_workflow(chat_request: ChatRequest) -> Workflow:
    # Access to request information for initialization
    return MyCustomWorkflow(chat_request.messages)

# Workflow input parameters (StartEvent):
# - user_msg: str - Current user message

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [run-llama/create-llama](https://github.com/run-llama/create-llama) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
