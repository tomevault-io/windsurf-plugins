---
trigger: always_on
description: This document provides context, patterns, and guidelines for AI coding assistants working in this repository. For human contributors, see [CONTRIBUTING.md](./CONTRIBUTING.md).
---

# AGENTS.md

This document provides context, patterns, and guidelines for AI coding assistants working in this repository. For human contributors, see [CONTRIBUTING.md](./CONTRIBUTING.md).

## Product Overview

Strands Agents is an open-source Python SDK for building AI agents with a model-driven approach. It provides a lightweight, flexible framework that scales from simple conversational assistants to complex autonomous workflows.

**Core Features:**
- Model Agnostic: Multiple model providers (Amazon Bedrock, Anthropic, OpenAI, Gemini, Ollama, etc.)
- Python-Based Tools: Simple `@tool` decorator with hot reloading
- MCP Integration: Native Model Context Protocol support
- Multi-Agent Systems: Agent-to-agent, swarms, and graph patterns
- Streaming Support: Real-time response streaming
- Hooks: Event-driven extensibility for agent lifecycle
- Session Management: Pluggable session managers (file, S3, custom)
- Observability: OpenTelemetry tracing and metrics

## Directory Structure

```
strands-agents/
│
├── src/strands/                          # Main package source code
│   ├── agent/                            # Core agent implementation
│   │   ├── agent.py                      # Main Agent class
│   │   ├── agent_result.py               # Agent execution results
│   │   ├── base.py                       # AgentBase protocol (agent interface)
│   │   ├── a2a_agent.py                  # A2AAgent client for remote A2A agents
│   │   ├── state.py                      # Agent state management
│   │   └── conversation_manager/         # Message history strategies
│   │       ├── conversation_manager.py           # Base conversation manager
│   │       ├── null_conversation_manager.py      # No-op manager
│   │       ├── sliding_window_conversation_manager.py  # Window-based
│   │       └── summarizing_conversation_manager.py     # Summarization-based
│   │
│   ├── event_loop/                       # Agent execution loop
│   │   ├── event_loop.py                 # Main loop logic
│   │   ├── streaming.py                  # Streaming response handling
│   │   └── _recover_message_on_max_tokens_reached.py
│   │
│   ├── models/                           # Model provider implementations
│   │   ├── model.py                      # Base model interface
│   │   ├── bedrock.py                    # Amazon Bedrock
│   │   ├── anthropic.py                  # Anthropic Claude
│   │   ├── openai.py                     # OpenAI
│   │   ├── gemini.py                     # Google Gemini
│   │   ├── ollama.py                     # Ollama local models
│   │   ├── litellm.py                    # LiteLLM unified interface
│   │   ├── mistral.py                    # Mistral AI
│   │   ├── llamaapi.py                   # LlamaAPI
│   │   ├── llamacpp.py                   # llama.cpp local
│   │   ├── sagemaker.py                  # AWS SageMaker
│   │   ├── writer.py                     # Writer AI
│   │   └── _validation.py                # Validation utilities
│   │
│   ├── tools/                            # Tool system
│   │   ├── decorator.py                  # @tool decorator
│   │   ├── tools.py                      # Tool base classes
│   │   ├── tool_provider.py              # ToolProvider interface
│   │   ├── registry.py                   # Tool registration
│   │   ├── loader.py                     # Dynamic tool loading
│   │   ├── watcher.py                    # Hot reload
│   │   ├── _caller.py                    # Tool invocation
│   │   ├── _validator.py                 # Tool validation
│   │   ├── _tool_helpers.py              # Helper utilities
│   │   ├── executors/                    # Tool execution environments
│   │   │   ├── _executor.py              # Base executor
│   │   │   ├── concurrent.py             # Thread/process pool
│   │   │   └── sequential.py             # Sequential execution
│   │   ├── mcp/                          # Model Context Protocol
│   │   │   ├── mcp_client.py             # MCP client implementation
│   │   │   ├── mcp_agent_tool.py         # MCP tool wrapper
│   │   │   ├── mcp_types.py              # MCP type definitions
│   │   │   ├── mcp_tasks.py              # Task-augmented execution config
│   │   │   └── mcp_instrumentation.py    # MCP telemetry
│   │   └── structured_output/            # Structured output handling
│   │       ├── structured_output_tool.py
│   │       ├── structured_output_utils.py
│   │       └── _structured_output_context.py
│   │
│   ├── multiagent/                       # Multi-agent patterns
│   │   ├── base.py                       # Base multi-agent classes
│   │   ├── graph.py                      # Graph-based orchestration
│   │   ├── swarm.py                      # Swarm pattern
│   │   ├── a2a/                          # Agent-to-agent protocol
│   │   │   ├── executor.py               # A2A executor
│   │   │   ├── server.py                 # A2A server
│   │   │   └── converters.py             # Strands/A2A type converters
│   │   └── nodes/                        # Graph node implementations
│   │
│   ├── types/                            # Type definitions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [strands-agents/sdk-python](https://github.com/strands-agents/sdk-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
