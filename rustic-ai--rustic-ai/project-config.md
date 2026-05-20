---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Rustic AI is a modular, multi-agent AI framework built around a "guild" architecture where agents collaborate on long-running tasks. The repository is a Poetry monorepo containing multiple packages that extend the core framework with integrations for LLMs, vector databases, web automation, and distributed execution.

## Build & Development Commands

### Setup
```bash
# Install Poetry plugin for monorepo support
poetry self add poetry-plugin-mono-repo-deps@0.3.2

# Install all dependencies including dev tools
poetry install --with dev --all-extras
```

### Testing
```bash
# Run all tests across all modules (requires Redis)
docker compose -f scripts/redis/docker-compose.yml up
poetry run tox

# Run tests for a specific file
poetry run pytest core/tests/utils/test_gemstone_id.py

# Test a new module individually before integration
cd <module-name>
poetry install --with dev --all-extras
poetry shell
tox
```

**Environment variables required for integration tests:**
- `OPENAI_API_KEY`
- `HUGGINGFACE_API_KEY`
- `SERP_API_KEY`

### Code Quality
```bash
# Format code
poetry run tox -e format
# Or directly:
isort .
black .

# Lint code
poetry run tox -e lint
# Or directly:
flake8
mypy .
```

### Building
```bash
# Build all modules
./scripts/poetry_install.sh
./scripts/poetry_build.sh

# Build Docker image (Linux)
docker build -t rusticai-api .

# Build Docker image (macOS)
docker buildx build --platform linux/amd64 -t rusticai-api .
```

## Architecture

### Core Concepts

**Guilds** (`core/src/rustic_ai/core/guild/guild.py`): Collections of agents that share a messaging bus, execution engine, and state. Each guild has a unique ID that namespaces all topics and prevents cross-guild message leakage.

**Agents** (`core/src/rustic_ai/core/guild/agent.py`): Task-specific AI workers that process messages via decorated handler methods. Agents are defined declaratively using `AgentSpec` and launched within guilds. Each agent subscribes to topics and uses `@processor` decorators to handle incoming messages.

**Messaging System** (`core/src/rustic_ai/core/messaging/`): Guild-scoped pub/sub message bus with pluggable backends (Redis for production, in-memory for testing). Messages use `GemstoneID` for globally sortable identifiers and maintain full routing history. Topics are automatically namespaced per guild (e.g., `"system"` becomes `"<guild-id>:system"`).

**Dependency Injection** (`core/src/rustic_ai/core/guild/agent_ext/depends/`): Agents declare dependencies via `DependencySpec` in their spec or via `AgentDependency` strings in `@processor` decorators. Dependencies can be scoped at agent, guild, or organization level. Common dependencies include LLMs, databases, filesystems, and knowledge bases.

**Execution Engines** (`core/src/rustic_ai/core/guild/execution/`): Manage agent lifecycle and runtime. Available engines: `SyncExecutionEngine` (single-threaded), `MultiThreadedExecutionEngine` (thread-pool), `MultiProcessExecutionEngine` (process-pool), and `RayExecutionEngine` (`ray/`) for distributed execution across clusters.

**Guild-to-Guild Communication** (`core/src/rustic_ai/core/guild/g2g/`): Cross-guild message routing via `GatewayAgent` and `EnvoyAgent`. Uses organization ID for shared namespace, enabling multi-guild workflows with session state preservation via saga pattern.

### Message Flow

1. Agent calls `context.send(payload)` in a processor method
2. `ProcessContext` resolves routing via `RoutingSlip` rules from guild spec
3. Message is persisted to backend (Redis sorted set scored by timestamp)
4. Backend publishes notification via pub/sub
5. `MessagingInterface` fans out to subscribed clients (excluding sender for self-inbox)
6. Clients push messages into priority heap sorted by `GemstoneID`
7. Target agent's processor is invoked with `ProcessContext`

Each message carries:
- `message_history`: List of `ProcessEntry` tracking agent hops
- `routing_slip`: Optional `RoutingSlip` defining multi-step workflows
- `session_state`: Dict for context passing between processors
- `thread`: List of message IDs forming a conversation chain

### Module Organization

The repository follows a consistent structure per module:
```
<module-name>/
  src/rustic_ai/<module-name>/     # Source code
  tests/                            # Tests using pytest
  pyproject.toml                    # Poetry config with dependencies
  tox.ini                           # Test automation config
  README.md                         # Module documentation
```

**Core modules:**
- `core/`: Base abstractions (Agent, Guild, Messaging, State)
- `redis/`: Redis messaging backend and state management
- `ray/`: Distributed execution engine
- `api/`: FastAPI server for guild management
- `testing/`: Test utilities (`wrap_agent_for_testing`, probe agents)

**LLM integrations:**
- `litellm/`: Unified LLM interface supporting 100+ providers
- `llm-agent/`: Pluggable LLM agent with tool calling
- `huggingface/`: HuggingFace models (diffusion, TTS, etc.)
- `vertexai/`: Google Vertex AI integration
- `claude/`: Claude integration via Vertex AI

**Data & tools:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rustic-ai/rustic-ai](https://github.com/rustic-ai/rustic-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
