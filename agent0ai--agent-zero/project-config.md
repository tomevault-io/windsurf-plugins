---
trigger: always_on
description: [Generated using reconnaissance on 2026-02-22]
---

# Agent Zero - AGENTS.md

[Generated using reconnaissance on 2026-02-22]

## Quick Reference
Tech Stack: Python 3.12+ | Flask | Alpine.js | LiteLLM | WebSocket (Socket.io)
Dev Server: python run_ui.py (runs on http://localhost:50001 by default)
Run Tests: pytest (standard) or pytest tests/test_name.py (file-scoped)
Documentation: README.md | docs/
Frontend Deep Dives: [Component System](docs/agents/AGENTS.components.md) | [Modal System](docs/agents/AGENTS.modals.md) | [Plugin Architecture](docs/agents/AGENTS.plugins.md) | [Banners & Discovery](docs/agents/AGENTS.banners.md)

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Core Commands](#core-commands)
3. [Docker Environment](#docker-environment)
4. [Project Structure](#project-structure)
5. [Development Patterns & Conventions](#development-patterns--conventions)
6. [Safety and Permissions](#safety-and-permissions)
7. [Code Examples](#code-examples)
8. [Git Workflow](#git-workflow)
9. [Release Notes](#release-notes)
10. [Troubleshooting](#troubleshooting)

---

## Project Overview

Agent Zero is a dynamic, organic agentic framework designed to grow and learn. It uses the operating system as a tool, featuring a multi-agent cooperation model where every agent can create subordinates to break down tasks.

Type: Full-Stack Agentic Framework (Python Backend + Alpine.js Frontend)
Status: Active Development
Primary Language(s): Python, JavaScript (ES Modules)

---

## Core Commands

### Setup
Do not combine these commands; run them individually:
```bash
pip install -r requirements.txt
pip install -r requirements2.txt
```
- Start WebUI: python run_ui.py

---

## Docker Environment

When running in Docker, Agent Zero uses two distinct Python runtimes to isolate the framework from the code being executed:

### 1. Framework Runtime (/opt/venv-a0)
- Version: Python 3.12.4
- Purpose: Runs the Agent Zero backend, API, and core logic.
- Packages: Contains all dependencies from requirements.txt.

### 2. Execution Runtime (/opt/venv)
- Version: Python 3.13
- Purpose: Default environment for the interactive terminal and the agent's code execution tool.
- Behavior: This is the environment active when you docker exec into the container. Packages installed by the agent via pip install during a task are stored here.

---

## Project Structure

```
/
├── agent.py              # Core Agent and AgentContext definitions
├── initialize.py         # Framework initialization logic
├── models.py             # LLM provider configurations
├── run_ui.py             # WebUI server entry point
├── api/                  # API Handlers (ApiHandler subclasses) + WsHandler subclasses (ws_*.py)
├── extensions/           # Backend lifecycle extensions
├── helpers/              # Shared Python utilities (plugins, files, etc.)
├── tools/                # Agent tools (Tool subclasses)
├── webui/
│   ├── components/       # Alpine.js components
│   ├── js/               # Core frontend logic (modals, stores, etc.)
│   └── index.html        # Main UI shell
├── usr/                  # User data directory (isolated from core)
│   ├── plugins/          # Custom user plugins
│   ├── settings.json     # User-specific configuration
│   └── workdir/          # Default agent workspace
├── plugins/              # Core system plugins
├── agents/               # Agent profiles (prompts and config)
├── prompts/              # System and message prompt templates
├── knowledge/
│   └── main/about/       # Agent self-knowledge (indexed into vector DB for runtime recall)
│       ├── identity.md           # Philosophy, principles, project context
│       ├── architecture.md       # Agent loop, memory pipeline, multi-agent, extensions
│       ├── capabilities.md       # Detailed capabilities and limitations
│       ├── configuration.md      # LLM roles, providers, profiles, plugins, settings
│       └── setup-and-deployment.md  # Docker deployment, updates, troubleshooting
└── tests/                # Pytest suite
```

Key Files:
- agent.py: Defines AgentContext and the main Agent class.
- helpers/plugins.py: Plugin discovery and configuration logic.
- webui/js/AlpineStore.js: Store factory for reactive frontend state.
- helpers/api.py: Base class for all API endpoints.
- scripts/openrouter_release_notes_system_prompt.md: Editable system prompt used to generate GitHub release notes during Docker publishing.
- knowledge/main/about/: Agent self-knowledge files, indexed into the vector DB for runtime recall. Not user-facing docs - written for the agent's internal reference.
- docs/agents/AGENTS.components.md: Deep dive into the frontend component architecture.
- docs/agents/AGENTS.modals.md: Guide to the stacked modal system.
- docs/agents/AGENTS.plugins.md: Comprehensive guide to the full-stack plugin system.

---

## Development Patterns & Conventions

### Backend (Python)
- Context Access: Use from agent import AgentContext, AgentContextType (not helpers.context).
- Communication: Use mq from helpers.messages to log proactive UI messages:
  mq.log_user_message(context.id, "Message", source="Plugin")
- API Handlers: Derive from ApiHandler in helpers/api.py.
- Extensions: Use the extension framework in helpers/extension.py for lifecycle hooks.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agent0ai/agent-zero](https://github.com/agent0ai/agent-zero) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
