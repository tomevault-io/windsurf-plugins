---
trigger: always_on
description: OS-APOW (Open Source Agentic Platform for Orchestration of Work) is a **Headless Agentic Orchestration Platform** that transforms interactive AI coding into autonomous background service. It converts GitHub Issues into "Execution Orders" that AI agents fulfill without human intervention, moving the agent from a passive co-pilot role to a background production service.
---

# AGENTS.md

## Project Overview

OS-APOW (Open Source Agentic Platform for Orchestration of Work) is a **Headless Agentic Orchestration Platform** that transforms interactive AI coding into autonomous background service. It converts GitHub Issues into "Execution Orders" that AI agents fulfill without human intervention, moving the agent from a passive co-pilot role to a background production service.

### Key Technologies

| Category | Technology | Version |
|----------|------------|---------|
| **Language** | Python | 3.12+ |
| **Package Manager** | uv | Latest |
| **Web Framework** | FastAPI | 0.115+ |
| **ASGI Server** | Uvicorn | 0.34+ |
| **Validation** | Pydantic | 2.10+ |
| **HTTP Client** | HTTPX | 0.28+ |
| **Testing** | pytest | 8.3+ |
| **Linting** | ruff | 0.8+ |
| **Type Checking** | mypy | 1.13+ |
| **Container** | Docker | Latest |

### Core Components

The system consists of four core pillars:

1. **The "Ear" (Notifier Service)** - FastAPI webhook receiver with HMAC SHA256 validation
2. **The State (Work Queue)** - GitHub Issues as database with label-based state machine
3. **The "Brain" (Sentinel Orchestrator)** - Async polling service for task management
4. **The "Hands" (Worker)** - DevContainer-based AI execution

---

## Setup Commands

### Prerequisites

- Python 3.12 or higher
- [uv](https://docs.astral.sh/uv/) package manager
- Docker (optional, for containerized deployment)
- GitHub token with appropriate permissions

### Installation

```bash
# Install dependencies with uv (recommended)
uv sync

# Or with pip
pip install -e .
```

### Environment Configuration

Set the required environment variables:

```bash
# Required
export GITHUB_TOKEN="your-github-token"
export GITHUB_ORG="your-org"
export GITHUB_REPO="your-repo"

# Optional
export SENTINEL_BOT_LOGIN="your-bot-login"  # For assign-then-verify locking
export WEBHOOK_SECRET="your-webhook-secret"  # For notifier service
```

### Running the Services

#### Sentinel Orchestrator (Polling Service)

```bash
# Using uv
uv run sentinel

# Or directly
python -m src.orchestrator_sentinel
```

#### Notifier Service (Webhook Receiver)

```bash
# Using uv
uv run notifier

# Or with uvicorn directly
uvicorn src.notifier_service:app --host 0.0.0.0 --port 8000
```

#### Using Docker Compose

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop services
docker-compose down
```

---

## Project Structure

```
workflow-orchestration-queue/
├── src/
│   ├── __init__.py
│   ├── orchestrator_sentinel.py      # Background polling and dispatch service
│   ├── notifier_service.py           # FastAPI webhook ingestion
│   ├── models/
│   │   ├── __init__.py
│   │   ├── work_item.py              # Unified WorkItem model with credential scrubbing
│   │   └── github_events.py          # GitHub webhook payload schemas
│   └── queue/
│       ├── __init__.py
│       └── github_queue.py           # ITaskQueue ABC + GitHubQueue implementation
├── tests/
│   ├── __init__.py
│   ├── unit/
│   │   ├── test_work_item.py         # Unit tests for WorkItem model
│   │   └── test_github_queue.py      # Unit tests for GitHubQueue
│   └── integration/
│       └── test_integration.py       # Integration tests
├── scripts/
│   ├── devcontainer-opencode.sh      # Core shell bridge for worker execution
│   └── gh-auth.ps1                   # GitHub App auth sync
├── docs/                              # Architecture and planning docs
├── plan_docs/                         # Planning documents
├── pyproject.toml                     # Project configuration and dependencies
├── Dockerfile                         # Application container definition
├── docker-compose.yml                 # Multi-service orchestration
└── README.md                          # Project documentation
```

### Key Files

| Path | Purpose |
|------|---------|
| `src/orchestrator_sentinel.py` | Sentinel orchestrator main entry point |
| `src/notifier_service.py` | FastAPI webhook receiver |
| `src/models/work_item.py` | Unified data model with credential scrubbing |
| `src/models/github_events.py` | GitHub webhook payload schemas |
| `src/queue/github_queue.py` | GitHub-backed task queue implementation |
| `pyproject.toml` | Project configuration, dependencies, and tool settings |
| `Dockerfile` | Application container definition |
| `docker-compose.yml` | Multi-service orchestration |

---

## Code Style

### Python Conventions

- **Python Version**: Target Python 3.12+
- **Line Length**: Maximum 100 characters (enforced by ruff)
- **Type Hints**: Use strict type checking with mypy
- **Imports**: Use absolute imports from `src` package
- **Docstrings**: Use triple-quoted docstrings for modules, classes, and public functions

### Linting and Formatting

The project uses **ruff** for both linting and formatting:

```bash
# Check for linting issues
uv run ruff check .

# Auto-fix linting issues
uv run ruff check --fix .

# Format code

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/intel-agency) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
