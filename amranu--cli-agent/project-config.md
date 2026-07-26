---
trigger: always_on
description: This document provides a comprehensive technical overview of the MCP Agent codebase, designed to help coding agents understand the system's sophisticated architecture, components, and core functionalities.
---

# AGENT.md

This document provides a comprehensive technical overview of the MCP Agent codebase, designed to help coding agents understand the system's sophisticated architecture, components, and core functionalities.

## 1. Overview

The MCP Agent is a sophisticated, extensible framework for creating language model-powered agents with advanced tool integration, subagent management, and multi-backend support. The codebase has evolved from a monolithic structure into a clean, modular architecture emphasizing maintainability, extensibility, and production-ready features.

### Core System Capabilities
- **Multi-LLM Backend Support:** Runtime-switchable support for 6 providers (Anthropic, OpenAI, DeepSeek, Google Gemini, OpenRouter, Ollama) with unified provider-model architecture
- **Advanced Subagent System:** Process-isolated subagents with event-driven communication and automatic result integration
- **Comprehensive Tool Ecosystem:** 16 built-in tools + external MCP protocol integration with unified execution pipeline
- **Context Management:** Intelligent context preservation with automatic conversation compaction and subagent delegation strategies
- **Professional UI/UX:** Event-driven terminal interface with streaming responses, interruption support, and slash commands
- **Production Features:** Exponential backoff retry logic, fault tolerance, persistent configuration management, and comprehensive error handling
- **Provider-Model Separation:** Same models accessible through different providers with flexible configuration

### Key Architectural Features

- **Provider-Model Architecture:** Clean separation between API providers and model characteristics via composition
- **Event-Driven Design:** Centralized event bus transforming streaming into discrete JSON events with interrupt support
- **Modular Core System:** 29 core modules with specialized responsibilities (base_agent, chat_interface, tool_execution_engine, etc.)
- **Strategy Pattern Implementation:** Interchangeable tool conversion and parsing strategies per LLM format
- **Context-Aware Operation:** Smart delegation of context-heavy tasks to preserve main agent efficiency
- **Stream-First Design:** Unified streaming interface across all backends with comprehensive interruption capabilities
- **Fault-Tolerant Design:** Comprehensive error handling, exponential backoff retry logic, and graceful degradation
- **Tool Extensibility:** Built-in tools (glob, grep, multiedit, etc.) + external MCP tool integration

## 2. Modular Architecture

The codebase implements a sophisticated modular architecture with clear separation of concerns and plugin-style extensibility:

```
cli_agent/
├── __init__.py                    # Package exports and version info
├── core/                          # Core agent framework (29 core files)
│   ├── base_agent.py              # Abstract base agent (2,122 lines)
│   ├── base_llm_provider.py       # Centralized LLM provider functionality (456 lines)
│   ├── base_provider.py           # Provider API abstraction (279 lines)
│   ├── model_config.py            # Model-specific configurations (531 lines)
│   ├── mcp_host.py                # Provider-model composition (607 lines)
│   ├── chat_interface.py          # Interactive chat management (912 lines)
│   ├── input_handler.py           # Terminal interaction utilities (303 lines)
│   ├── slash_commands.py          # Command system management (806 lines)
│   ├── tool_execution_engine.py   # Tool execution and validation (280 lines)
│   ├── builtin_tool_executor.py   # Tool implementations (685 lines)
│   ├── event_system.py            # Central event bus architecture (523 lines)
│   ├── display_manager.py         # Event-driven display coordination (400 lines)
│   ├── response_handler.py        # Response processing framework (682 lines)
│   ├── subagent_coordinator.py    # Subagent lifecycle management
│   ├── tool_permissions.py        # Tool access control system (421 lines)
│   └── [15 additional specialized modules]
├── providers/                     # API provider implementations
│   ├── anthropic_provider.py      # Anthropic API provider
│   ├── openai_provider.py         # OpenAI API provider
│   ├── deepseek_provider.py       # DeepSeek API provider
│   ├── google_provider.py         # Google Gemini API provider
│   ├── openrouter_provider.py     # OpenRouter multi-model API
│   └── ollama_provider.py         # Ollama local inference provider
├── tools/                         # Built-in tool definitions
│   ├── __init__.py                # Tool exports
│   └── builtin_tools.py           # 16 built-in tools with JSON schemas (475 lines)
├── utils/                         # Shared utility modules (7 utility modules)
│   ├── __init__.py                # Centralized utility exports
│   ├── tool_conversion.py         # Multi-format tool schema conversion
│   ├── tool_parsing.py            # Tool call parsing from LLM responses
│   ├── content_processing.py      # Content extraction and cleaning utilities
│   ├── http_client.py             # HTTP client factory and lifecycle management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amranu/cli-agent](https://github.com/amranu/cli-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
