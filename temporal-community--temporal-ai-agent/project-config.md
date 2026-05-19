---
trigger: always_on
description: - `workflows/` - Temporal workflows including the main AgentGoalWorkflow for multi-turn AI conversations
---

# Temporal AI Agent Contribution Guide

## Repository Layout
- `workflows/` - Temporal workflows including the main AgentGoalWorkflow for multi-turn AI conversations
- `activities/` - Temporal activities for tool execution and LLM interactions  
- `tools/` - Native AI agent tool implementations organized by category (finance, HR, ecommerce, travel, etc.)
- `goals/` - Agent goal definitions organized by category, supporting both native and MCP tools
- `shared/` - Shared configuration including MCP server definitions
- `models/` - Data types and tool definitions used throughout the system
- `prompts/` - Agent prompt generators and templates
- `api/` - FastAPI server that exposes REST endpoints to interact with workflows
- `frontend/` - React-based web UI for chatting with the AI agent
- `tests/` - Comprehensive test suite for workflows and activities using Temporal's testing framework
- `enterprise/` - .NET worker implementation for enterprise activities (train booking)
- `scripts/` - Utility scripts for running workers and testing tools

## Running the Application

### Quick Start with Docker
```bash
# Start all services with development hot-reload
docker compose up -d

# Quick rebuild without infrastructure
docker compose up -d --no-deps --build api worker frontend
```

Default URLs:
- Temporal UI: http://localhost:8080
- API: http://localhost:8000  
- Frontend: http://localhost:5173

### Local Development Setup

1. **Prerequisites:**
   ```bash
   # Install uv and Temporal server (MacOS)
   brew install uv
   brew install temporal

   temporal server start-dev
   ```

2. **Backend (Python):**
   ```bash
   # Quick setup using Makefile
   make setup              # Creates venv and installs dependencies
   make run-worker         # Starts the Temporal worker
   make run-api            # Starts the API server
   
   # Or manually:
   uv sync
   uv run scripts/run_worker.py    # In one terminal
   uv run uvicorn api.main:app --reload   # In another terminal
   ```

3. **Frontend (React):**
   ```bash
   make run-frontend       # Using Makefile
   
   # Or manually:
   cd frontend
   npm install
   npx vite
   ```

4. **Enterprise .NET Worker (optional):**
   ```bash
   make run-enterprise     # Using Makefile
   
   # Or manually:
   cd enterprise
   dotnet build
   dotnet run
   ```

### Environment Configuration
Copy `.env.example` to `.env` and configure:
```bash
# Required: LLM Configuration
LLM_MODEL=openai/gpt-4o
LLM_KEY=your-api-key-here
# LLM_MODEL=anthropic/claude-3-5-sonnet-20240620
# LLM_KEY=${ANTHROPIC_API_KEY}
# LLM_MODEL=gemini/gemini-2.5-flash-preview-04-17
# LLM_KEY=${GOOGLE_API_KEY}

# Optional: Agent Goals and Categories  
AGENT_GOAL=goal_choose_agent_type
GOAL_CATEGORIES=hr,travel-flights,travel-trains,fin,ecommerce,mcp-integrations,food

# Optional: Tool-specific APIs
STRIPE_API_KEY=sk_test_...       # For invoice creation
# `goal_event_flight_invoice` works without this key – it falls back to a mock invoice if unset
FOOTBALL_DATA_API_KEY=...        # For real football fixtures
```

## Testing

The project includes comprehensive tests using Temporal's testing framework:

```bash
# Install test dependencies
uv sync

# Run all tests
uv run pytest

# Run with time-skipping for faster execution  
uv run pytest --workflow-environment=time-skipping

# Run specific test categories
uv run pytest tests/test_tool_activities.py -v     # Activity tests
uv run pytest tests/test_agent_goal_workflow.py -v # Workflow tests

# Run with coverage
uv run pytest --cov=workflows --cov=activities
```

**Test Coverage:**
- ✅ **Workflow Tests**: AgentGoalWorkflow signals, queries, state management
- ✅ **Activity Tests**: ToolActivities, LLM integration (mocked), environment configuration  
- ✅ **Integration Tests**: End-to-end workflow and activity execution

**Documentation:**
- **Quick Start**: [testing.md](docs/testing.md) - Simple commands to run tests
- **Comprehensive Guide**: [tests/README.md](tests/README.md) - Detailed testing patterns and best practices

## Linting and Code Quality

```bash
# Using poe tasks
uv run poe format    # Format code with black and isort
uv run poe lint      # Check code style and types
uv run poe test      # Run test suite

# Manual commands
uv run black .
uv run isort .
uv run mypy --check-untyped-defs --namespace-packages .
```

## Agent Customization

### Adding New Goals and Tools

#### For Native Tools:
1. Create tool implementation in `tools/` directory
2. Add tool function mapping in `tools/__init__.py`  
3. Register tool definition in `tools/tool_registry.py`
4. Add tool names to static tools list in `workflows/workflow_helpers.py`
5. Create or update goal definition in appropriate file in `goals/` directory

#### For MCP Tools:
1. Configure MCP server definition in `shared/mcp_config.py` (for reusable servers)
2. Create or update goal definition in appropriate file in `goals/` directory with `mcp_server_definition`
3. Set required environment variables (API keys, etc.)

#### For Goals:
1. Create goal file in `goals/` directory (e.g., `goals/my_category.py`)
2. Import and extend the goal list in `goals/__init__.py`

### Configuring Goals
The agent supports multiple goal categories organized in `goals/`:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [temporal-community/temporal-ai-agent](https://github.com/temporal-community/temporal-ai-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
