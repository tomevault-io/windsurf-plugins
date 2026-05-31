---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**AskUI Vision Agent** is a Python desktop and mobile automation framework that enables AI agents to control computers (Windows, macOS, Linux), mobile devices (Android, iOS), and HMI systems. It supports both programmatic UI automation (RPA-like single-step commands) and agentic intent-based instructions using vision/computer vision models.

**Tech Stack:** Python 3.10+, Pydantic 2, Anthropic SDK, OpenTelemetry, Model Context Protocol (MCP), PDM

## Common Commands

### Development Setup
```bash
# Install dependencies
pdm install
```

### Testing
```bash
# Run all tests (parallel execution)
pdm run test

# Run specific test suites
pdm run test:unit          # Unit tests only
pdm run test:integration   # Integration tests only
pdm run test:e2e          # End-to-end tests only

# Run tests with coverage
pdm run test:cov          # All tests with coverage report
pdm run test:cov:view     # View coverage report in browser
```

### Code Quality
```bash
# Quick QA: type check, format, and fix linting issues (run before commits)
pdm run qa:fix

# Individual commands
pdm run typecheck:all     # Type checking with mypy
pdm run format            # Format code with ruff
pdm run lint              # Lint code with ruff
pdm run lint:fix          # Auto-fix linting issues
```

### Code Generation
```bash
# Regenerate gRPC client code from .proto files
pdm run grpc:gen

# Regenerate Pydantic models from JSON schemas
pdm run json:gen
```

## High-Level Architecture

### Core SDK Architecture

```
ComputerAgent (Main SDK Entry Point)
    ↓
Agent (Abstract base class for all agents)
    ├── ComputerAgent (Desktop automation)
    ├── AndroidAgent (Mobile Android automation)
    ├── WebVisionAgent (Web-specific automation)
    └── WebTestingAgent (Web testing framework)

    Uses:
    ├── ModelRouter → Model selection/composition
    ├── AgentToolbox → Tool & OS abstraction
    └── Locators → UI element identification
```

**Key Flow:**
1. User calls `agent.click("Submit button")` on `ComputerAgent`
2. `AgentBase.locate()` routes to appropriate model via `ModelRouter`
3. Model receives screenshot + locator → returns coordinates
4. `AgentToolbox.os.click()` → gRPC call to Agent OS
5. Agent OS performs actual mouse click

### Chat API Architecture

```
FastAPI Chat API (Experimental)
    ├── Assistants (AI agent configurations)
    ├── Threads (Conversation sessions)
    ├── Messages (Chat history)
    ├── Runs (Agent execution iterations)
    ├── Files (Attachments & resources)
    ├── MCP Configs (Tool providers)
    └── Workflows & Scheduled Jobs (Automation triggers)
```

**Key Flow:**
1. User → Chat UI (hub.askui.com) → Chat API (FastAPI)
2. Thread/Messages stored in SQLAlchemy database
3. Runs execute agent steps in a loop
4. Agent uses ModelRouter → Tools (MCP servers or direct) → AgentOS

### Model Router & Composition

The `ModelRouter` provides a flexible abstraction for AI model selection:

```python
# Single model for all tasks
model = "askui"

# Task-specific models (ActModel, GetModel, LocateModel)
model = {
    "act": "claude-sonnet-4-20250514",
    "get": "askui",
    "locate": "askui-combo"
}

# Custom registry
models = ModelRegistry()
models.register("my-model", custom_model_instance)
```

**Supported Model Providers:**
- **AskUI Models** (Primary - internally hosted)
- **Anthropic Claude** (Computer Use, Messages API)
- **Google Gemini** (via OpenRouter)
- **Hugging Face Spaces** (Community models)

### Agent OS Abstraction

`AgentOs` provides an abstraction layer for OS-level operations:

```
AgentOs (Abstract Interface)
    ├── AskUiControllerClient (gRPC to AskUI Agent OS - primary)
    ├── PlaywrightAgentOs (Web browser automation)
    └── AndroidAgentOs (Android ADB)
```

### Locator System

Locators identify UI elements in multiple ways:

- **Text**: Match by text content (exact/similar/contains/regex)
- **Image**: Match by image file or base64
- **Prompt**: Natural language description
- **Coordinate**: Absolute (x, y) position
- **Relatable**: Positional relationships (right_of, below, etc.)

Serialization differs by model type (VLM vs. traditional).

### Tool System (MCP)

Tools follow the Model Context Protocol (MCP) for extensibility:

```
Tools (MCP Servers)
    ├── Computer: screenshot, click, type, mouse, clipboard
    ├── Android: device control via ADB
    ├── Testing: scenario & feature management
    └── Utility: file ops, data extraction
```

Tools are auto-discovered and can be dynamically loaded via MCP configurations.

## Key Code Locations

### Core SDK
- `src/askui/agent.py` - Main `ComputerAgent` class (user-facing API)
- `src/askui/agent_base.py` - Abstract `Agent` (base) with shared agent logic
- `src/askui/android_agent.py` - Android-specific agent
- `src/askui/web_agent.py` - Web-specific agent

### Models & AI
- `src/askui/models/` - AI model providers & router factory
- `src/askui/models/shared/` - Shared abstractions (`Agent`, `Tool`, `MessagesApi`)
- `src/askui/models/{provider}/` - Provider implementations
- `src/askui/prompts/` - System prompts for different models

### Tools & OS

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [askui/python-sdk](https://github.com/askui/python-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
