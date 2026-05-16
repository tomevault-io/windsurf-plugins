---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---


# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Testing
```bash
poetry run pytest                    # Run all tests
poetry run pytest tests/unit/        # Run unit tests only
poetry run pytest tests/integration/ # Run integration tests only
poetry run pytest --cov=arshai      # Run tests with coverage
```

### Code Quality
```bash
poetry run black .                  # Format code
poetry run isort .                  # Sort imports
poetry run mypy arshai/             # Type checking
poetry run bandit -r arshai/        # Security analysis
poetry run safety check             # Check dependencies for vulnerabilities
```

### Documentation
```bash
cd docs_sphinx && make html  # Build Sphinx documentation
```

### Package Management
```bash
poetry install           # Install all dependencies
poetry install -E all    # Install with all optional dependencies (redis, milvus, flashrank)
poetry install -E docs   # Install documentation dependencies
```

## Architecture Overview

Arshai is an AI application framework built on **clean architecture principles** with **interface-driven design**. The system follows a layered approach with clear separation of concerns:

### Core Layers

1. **Application Layer**: Workflows and Agents that orchestrate business logic
2. **Domain Layer**: Memory, Tools, and LLM integrations that handle core functionality  
3. **Infrastructure Layer**: Document processing, vector storage, and external integrations

### Key Components

- **Agents** (`arshai/agents/`): Intelligence units that process user interactions
- **Workflows** (`arshai/workflows/`): Orchestrate complex multi-step processes with state management
- **Memory** (`arshai/memory/`): Handle conversation context and state persistence
- **Tools** (`arshai/tools/`): External capabilities that extend agent functionality
- **LLMs** (`arshai/llms/`): Language model integrations with unified interface
- **Factories** (`arshai/factories/`): Component creation and dependency injection

### Design Patterns

- **Interface-First**: All major components implement well-defined protocols in `arshai/core/interfaces/`
- **Factory Pattern**: Component creation abstracted through factory classes
- **DTO Pattern**: Structured data transfer using Pydantic models
- **Provider Pattern**: Multiple implementations for external services (LLMs, memory, vector DBs)
- **Async-First**: Most operations support asynchronous execution

## Development Guidelines

### Working with the Codebase

1. **Start with Interfaces**: Examine contracts in `arshai/core/interfaces/` before implementations
2. **Use Factories**: Leverage existing factory classes in `arshai/factories/` for component creation
3. **Follow DTO Pattern**: Use structured Pydantic models for all data interactions
4. **Prefer Async**: Use async methods for better performance and concurrency
5. **Maintain Immutability**: Especially important for workflow state management

### Project Structure

- **Main Package**: `arshai/` - New unified structure (use this for new development)
- **Legacy Code**: `src/` - Being migrated (avoid for new features)
- **Core Interfaces**: `arshai/core/interfaces/` - System contracts and protocols
- **Configuration**: `arshai/config/` - Settings and configuration management
- **Examples**: `examples/` - Working code samples and usage patterns

### Key Interface Locations

- **IAgent**: Core agent contract for user interactions
- **IWorkflowOrchestrator/IWorkflowRunner**: Workflow system contracts
- **IMemoryManager**: Memory management interface
- **ILLM**: Language model provider interface
- **IVectorDBClient/IEmbedding**: Vector storage and embeddings

### Extension Points

- **Custom Agents**: Implement `IAgent` interface
- **Custom Tools**: Create callable functions with proper type hints and docstrings
- **LLM Providers**: Implement `ILLM` interface
- **Memory Backends**: Implement `IMemoryManager` interface
- **Workflow Nodes**: Extend base node classes for business logic

### Common Patterns

```python
# Component creation via factories
settings = Settings()
agent = settings.create_agent("conversation", agent_config)

# Structured input/output
input_data = IAgentInput(message="...", conversation_id="...")
response, usage = await agent.process_message(input_data)

# Tool integration (function-based pattern)
def search_web(query: str) -> str:
    """Search the web for information."""
    # Tool implementation
    return "search results"

def query_knowledge_base(question: str) -> str:
    """Query the knowledge base."""
    # Tool implementation
    return "knowledge base answer"

# Background tasks (fire-and-forget execution)
def notify_admin(event: str, details: str = ""):
    """Background task that runs independently."""
    print(f"Admin notification: {event} - {details}")

# Use tools with LLM
regular_functions = {
    "search_web": search_web,
    "query_knowledge_base": query_knowledge_base
}
background_tasks = {"notify_admin": notify_admin}

llm_input = ILLMInput(
    system_prompt="...",
    user_message="...",
    regular_functions=regular_functions,
    background_tasks=background_tasks
)

# Workflow orchestration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [felesh-ai/arshai](https://github.com/felesh-ai/arshai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
