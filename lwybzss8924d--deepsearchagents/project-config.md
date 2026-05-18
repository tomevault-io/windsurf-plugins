---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## DeepSearchAgent Project

DeepSearchAgent is an intelligent agent system that combines the ReAct (Reasoning + Acting) framework and the CodeAct concept (executable code agents) to enable deep web search and reasoning capabilities. Built on Hugging Face's `smolagents` framework, it provides multi-step reasoning for complex queries through various interfaces.

**Version 0.3.3.dev** introduces a complete web frontend with real-time streaming, metadata-driven component routing, and a simplified WebSocket API architecture.

## Development Commands

### Common Development Tasks
```bash
# Install dependencies (with development tools)
uv pip install -e ".[dev,test,cli]"

# Run tests
make test

# Run specific test file
uv run -- pytest tests/test_specific.py

# Run tests with coverage
uv run -- pytest --cov=src tests

# Start development servers
make run        # FastAPI server (default port 8000)
# Web API v2 is accessible via the main FastAPI server
make cli        # Interactive CLI mode

# CLI with specific agent modes
make cli-react  # ReAct agent (non-interactive)
make cli-codact # CodeAct agent (non-interactive)

# MCP server
python -m src.agents.servers.run_fastmcp

# Frontend development
cd frontend && npm install   # Install dependencies
cd frontend && npm run dev   # Start development server (port 3000)
cd frontend && npm run build # Build for production
cd frontend && npm run lint  # Run linting
```

### Configuration Setup
```bash
cp config.template.toml config.toml  # Non-sensitive settings
cp .env.example .env                 # API keys
```

## High-Level Architecture

### Agent System Design
The project implements a dual-agent architecture:
- **CodeAct Agent**: Executes Python code to perform actions, suitable for computational tasks
- **ReAct Agent**: Uses structured tool calling with JSON responses, better for web search tasks
- **Runtime Manager**: Handles execution environment and tool availability for both agent types
- **Model Routing**: Supports multiple LLM providers through LiteLLM with configurable model selection
- **Web API v2**: Simplified WebSocket API with direct Gradio message pass-through (~500 lines, down from ~5000)

### Core Components Interaction
```
User Input → Agent Selection → Planning/Reasoning → Tool Execution → Response Streaming
                    ↓                    ↓                ↓
              config.toml          Prompt Templates    Toolbox Manager
                    
Web API v2 → WebSocket → Session Manager → Gradio Passthrough → stream_to_gradio → Agent
                           │
                           ▼
                    Frontend (Next.js)
                           │
                    ┌──────┴──────┐
                    │  WebSocket  │
                    │    Hook     │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │  Component  │
                    │   Router    │
                    └──────┬──────┘
                           │
         ┌─────────────────┼─────────────────┐
         │                 │                 │
    ┌────▼────┐      ┌────▼────┐      ┌────▼────┐
    │Planning │      │ Action  │      │ Final   │
    │  Card   │      │ Thought │      │ Answer  │
    └─────────┘      └─────────┘      └─────────┘
```

### Tool Ecosystem
Tools are unified through a common interface supporting both sync/async operations:
- **Search Tools**: Multi-engine support (Google via Serper, X.com via xAI)
- **Content Processing**: URL reading, text chunking, embedding, reranking
- **Computation**: WolframAlpha integration for mathematical queries
- **Final Answer**: Structured response generation with source attribution

### Configuration Hierarchy
1. Command-line arguments (highest priority)
2. Environment variables
3. config.toml settings
4. Default values (lowest priority)

### Streaming Architecture
- Streaming is now available but disabled by default for stability
- When enabled, provides real-time visibility into agent reasoning process
- Supports both FastAPI SSE and Gradio streaming interfaces
- CLI now supports streaming output with the new StreamingConsoleFormatter

#### Enabling Streaming
To enable streaming output in the CLI:

1. **In config.toml**:
```toml
[agents.common]
cli_streaming_enabled = true  # Global toggle for CLI streaming

[agents.react]
enable_streaming = true  # Enable for React agent

[agents.codact]
enable_streaming = true  # Enable for CodeAct agent
```

2. **Via command line**:
```bash
# Enable streaming for a single query
python -m src.cli --agent-type react --enable-streaming --query "your query"
```

3. **Environment variables** (highest priority):
```bash
export REACT_ENABLE_STREAMING=true
export CODACT_ENABLE_STREAMING=true
export CLI_STREAMING_ENABLED=true
```

Note: Streaming support depends on the agent and model capabilities. Not all models support streaming output.

### Web API v2 Design Principles

The v2 API follows a simplified architecture:
1. **Direct Pass-through**: Messages from smolagents' stream_to_gradio are passed with minimal transformation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lwyBZss8924d/DeepSearchAgents](https://github.com/lwyBZss8924d/DeepSearchAgents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
