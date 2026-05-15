---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**AgentShip** is a production-ready framework for building, deploying, and operating AI agents. Built on Google ADK and LangGraph, it provides REST API, session management, observability, streaming, and deployment tools.

**Key Technologies:**
- Python 3.13+ with pipenv for dependency management
- FastAPI (REST API server on port 7001)
- Google ADK & LangGraph (dual execution engines)
- PostgreSQL (session storage)
- LiteLLM (multi-provider LLM support)
- Docker & Docker Compose (containerization)
- OPIK (observability)
- MCP (Model Context Protocol for tool integration)
- Auto Tool Documentation (automatic prompt generation from schemas)

## Commands

### Docker Development (Recommended)
```bash
make docker-setup    # First-time setup (creates .env, builds, starts containers)
make docker-up       # Start containers (API: localhost:7001)
make docker-down     # Stop containers
make docker-restart  # Quick restart
make docker-reload   # Hard reload (rebuild + restart)
make docker-logs     # View logs
```

**Ports:**
- API/Swagger/Docs/AgentShip Studio: `localhost:7001`
- PostgreSQL (external): `localhost:5433` (inside Docker: `postgres:5432`)

### Local Development (No Docker)
```bash
make dev             # Start dev server (localhost:7001)
pipenv run uvicorn src.service.main:app --reload --host 0.0.0.0 --port 7001
```

### Testing
```bash
make test            # Run all tests
make test-cov        # Run tests with coverage report
pipenv run pytest tests/ -v
pipenv run pytest tests/unit/ -v           # Unit tests only
pipenv run pytest tests/integration/ -v    # Integration tests only
pipenv run pytest tests/unit/test_file.py -v  # Single test file
```

### Code Quality
```bash
make lint            # Run flake8 linter
make format          # Format code with black
make type-check      # Run mypy type checking
```

### Documentation
```bash
make docs-build      # Build Sphinx documentation
make docs-serve      # Build docs + start server (localhost:7001/docs)
```

### Deployment
```bash
make heroku-deploy   # Deploy to Heroku (one command)
```

### Utilities
```bash
make clean           # Clean caches (__pycache__, .pytest_cache, etc.)
pipenv install       # Install production dependencies
pipenv install --dev # Install development dependencies
```

## Architecture

### Directory Structure
```
src/
├── agent_framework/      # Core framework (engines, tools, sessions, config)
│   ├── configs/          # Agent & LLM configuration loaders
│   ├── core/             # BaseAgent, I/O, types
│   ├── engines/          # ADK & LangGraph execution engines
│   │   ├── adk/          # Google ADK engine
│   │   └── langgraph/    # LangGraph engine with LiteLLM
│   ├── factories/        # Engine, memory, observability, tool factories
│   ├── mcp/              # MCP (Model Control Plane) integration
│   ├── middleware/       # Middleware system
│   ├── observability/    # OPIK observability integration
│   ├── registry/         # Agent discovery and registration
│   ├── session/          # Session storage (ADK, LangGraph adapters)
│   ├── tools/            # Tool system (adapters, base tool, domains)
│   └── utils/            # Utilities (path, PDF, Azure)
├── all_agents/           # Agent implementations (auto-discovered)
│   ├── base_agent.py     # Public import surface (BaseAgent, AgentType)
│   ├── single_agent_pattern/
│   ├── orchestrator_pattern/
│   ├── tool_pattern/
│   └── ...
├── service/              # FastAPI REST API layer
│   ├── main.py           # FastAPI app + routes
│   └── routers/          # API routers (REST, SSE, WebSocket)
└── log_settings.py       # Logging configuration

tests/
├── unit/                 # Unit tests
└── integration/          # Integration tests

debug_ui/                 # AgentShip Studio (localhost:7001/studio)
docs_sphinx/              # Sphinx documentation source
agent_store_deploy/       # Database setup scripts
service_cloud_deploy/     # Heroku deployment scripts
```

### Core Concepts

#### 1. Agent Definition (YAML + Python)
Agents are defined using two files:
- **main_agent.yaml**: Configuration (LLM, temperature, instructions, tools, MCP)
- **main_agent.py**: Python class inheriting from `BaseAgent`

**Example: Creating a New Agent**
```bash
# 1. Create directory
mkdir -p src/all_agents/my_agent

# 2. Create main_agent.yaml
cat > src/all_agents/my_agent/main_agent.yaml << EOF
agent_name: my_agent
llm_provider_name: openai
llm_model: gpt-4o
temperature: 0.4
execution_engine: adk  # or langgraph
streaming_mode: none   # or token_by_token
description: My helpful assistant
instruction_template: |
  You are a helpful assistant.
EOF

# 3. Create main_agent.py
cat > src/all_agents/my_agent/main_agent.py << EOF
from src.all_agents.base_agent import BaseAgent
from src.service.models.base_models import TextInput, TextOutput
from src.agent_framework.utils.path_utils import resolve_config_path

class MyAgent(BaseAgent):
    def __init__(self):
        super().__init__(
            config_path=resolve_config_path(relative_to=__file__),
            input_schema=TextInput,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Agent-Ship/agent-ship](https://github.com/Agent-Ship/agent-ship) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
