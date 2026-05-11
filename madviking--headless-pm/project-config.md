---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Environment Setup

### Automated Setup (Recommended)
```bash
# Run universal setup script - handles platform-specific requirements
./setup/universal_setup.sh

# This will:
# - Detect your architecture (ARM64 for native Mac, x86_64 for Claude Code)
# - Create the appropriate virtual environment (venv or claude_venv)
# - Install correct package versions for your platform
# - Create .env from env-example if needed
```

### Manual Setup (if needed)
```bash
# For LLM agents (x86_64), use claude_venv:
python -m venv claude_venv
source claude_venv/bin/activate
pip install pydantic==2.11.7 pydantic-core==2.33.2
pip install -r setup/requirements.txt

# For native Mac (ARM64), use standard venv:
python -m venv venv
source venv/bin/activate
pip install -r setup/requirements.txt

# Configure environment
cp env-example .env
# Edit .env with API keys and database configuration

# Initialize database
python -m src.cli.main init
python -m src.cli.main seed  # Optional: add sample data
```

### Running the Application

#### Quick Start (Recommended)
```bash
# Activate your virtual environment first
source venv/bin/activate  # or source claude_venv/bin/activate

# Use the automated start script (checks environment, DB, starts server)
./start.sh
```

#### Manual Start
```bash
# Run API server
bash start.sh
```

The `start.sh` script automatically:
- ✅ Validates Python 3.11+ requirement  
- ✅ Checks required packages are installed
- ✅ Creates .env from env-example if needed
- ✅ Tests database connection
- ✅ Initializes database if needed
- ✅ Checks port availability
- ✅ Starts server with proper configuration
- ✅ Starts only services with defined ports in .env

**Service Port Configuration:**
- Services are only started if their port is defined in `.env`
- To skip a service, remove or comment out its port variable:
  - `SERVICE_PORT` - API server (default: 6969)
  - `MCP_PORT` - MCP server (default: 6968) 
  - `DASHBOARD_PORT` - Web dashboard (default: 3001)

**Note**: Activate your virtual environment before running the start script.

## Testing

### Running Tests

**IMPORTANT**: Full test suite requires **at least 3 minutes** to complete (155 tests in ~2.5 min)

```bash
# IMPORTANT: Use the appropriate virtual environment for your platform
# For Claude Code (x86_64):
source claude_venv/bin/activate

# For native Mac (ARM64):
source venv/bin/activate

# Run all tests with coverage (requires 3+ minutes)
timeout 300s python -m pytest --cov=src --cov-report=term-missing

# Quick unit tests only (~30 seconds)
python -m pytest tests/unit/ -v

# Run specific test files
python -m pytest tests/test_api.py -v
python -m pytest tests/test_models.py -v

# Run tests without coverage (faster)
python -m pytest -q

# Run specific test patterns
python -m pytest -k "test_name_pattern"

# View test durations to identify slow tests
python -m pytest -v --durations=20
```

### Test Coverage Status
- **Current Tests**: Client integration tests
- **Test Location**: `tests/test_headless_pm_client.py`
- **Additional Testing**: Comprehensive test suite planned

### Test Architecture Notes
- API tests use temporary file-based SQLite databases (not in-memory) to avoid connection issues
- All models, enums, and services have comprehensive test coverage
- Tests include document creation, mention detection, service registry, and task lifecycle

## Architecture Overview

Headless PM is a REST API for LLM agent task coordination with document-based communication. Key architectural decisions:

1. **Document-Based Communication**: Agents communicate via documents with @mention support
2. **Service Registry**: Track running services with heartbeat monitoring
3. **Git Workflow Integration**: Major tasks use feature branches with PRs, minor tasks commit directly to main
4. **Changes Polling**: Efficient polling endpoint for agents to get updates since last check
5. **Role-Based System**: Five roles (frontend_dev, backend_dev, qa, architect, pm) with multiple agents per role
6. **Task Complexity**: Major/minor classification determines Git workflow (PR vs direct commit)
7. **Comprehensive Testing**: 78 tests with 71% coverage including full API testing

### Enhanced Features
- **Epic/Feature/Task Hierarchy**: Three-level project organization
- **Documents Table**: Agent communication with mention detection
- **Service Registry**: Track microservices with heartbeat monitoring and ping URLs
- **Mentions System**: @username notifications across documents and tasks
- **Changes API**: Polling endpoint returning changes since timestamp
- **Task Complexity**: Major/minor enum driving branching strategies
- **Connection Types**: Distinguish between MCP and client connections
- **Task Comments**: Collaborative discussion on tasks with @mentions
- **Python Client Helper**: Complete CLI interface (`headless_pm_client.py`)
- **MCP Server**: Natural language interface for Claude Code
- **Database Migrations**: Schema evolution support
- **Web Dashboard**: Real-time project overview with analytics and monitoring
- **Port-based Service Control**: Services only start if their port is defined in .env


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madviking/headless-pm](https://github.com/madviking/headless-pm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
