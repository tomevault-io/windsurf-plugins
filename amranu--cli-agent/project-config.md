---
trigger: always_on
description: This document provides comprehensive technical documentation for the MCP Agent codebase, specifically designed to help Claude Code and other AI assistants understand the modular architecture, components, and development patterns.
---

# CLAUDE.md - MCP Agent Architecture Documentation

This document provides comprehensive technical documentation for the MCP Agent codebase, specifically designed to help Claude Code and other AI assistants understand the modular architecture, components, and development patterns.

## 🏗️ Architecture Overview

The MCP Agent has been refactored from a monolithic 3,237-line file into a clean modular architecture with a sophisticated Provider-Model system. The system provides an extensible framework for creating language model-powered agents with tool integration, interactive chat, and multi-backend support via a flexible provider-model architecture.

### Key Architectural Principles

- **Modular Design**: Components separated by responsibility
- **Provider-Model Separation**: API providers separated from model characteristics
- **Model Agnosticism**: Core logic independent of LLM implementation  
- **Tool Extensibility**: Support for both built-in and MCP protocol tools
- **Interactive & Programmatic**: CLI and library usage patterns
- **Centralized Management**: Single source of truth for shared functionality
- **Multi-Provider Support**: Same model accessible through different providers

## 📁 Modular File Structure

```
agent/
├── cli_agent/                    # Main package - modular architecture
│   ├── __init__.py              # Package exports and version
│   ├── core/                    # Core agent functionality (29 core files)
│   │   ├── __init__.py         # Core component exports
│   │   ├── base_agent.py       # BaseMCPAgent abstract class (2,122 lines)
│   │   ├── base_llm_provider.py # BaseLLMProvider shared functionality (456 lines)
│   │   ├── base_provider.py    # BaseProvider API abstraction (279 lines)
│   │   ├── model_config.py     # ModelConfig classes for LLM characteristics (531 lines)
│   │   ├── mcp_host.py         # MCPHost unified provider+model interface (607 lines)
│   │   ├── chat_interface.py   # Interactive chat management (912 lines)
│   │   ├── input_handler.py    # InterruptibleInput terminal handling (303 lines)
│   │   ├── slash_commands.py   # SlashCommandManager command system (806 lines)
│   │   ├── tool_execution_engine.py # Tool execution and validation (280 lines)
│   │   ├── builtin_tool_executor.py # Built-in tool implementations (685 lines)
│   │   ├── event_system.py     # Central event bus architecture (523 lines)
│   │   ├── display_manager.py  # Event-driven display coordination (400 lines)
│   │   ├── response_handler.py # Response processing framework (682 lines)
│   │   ├── subagent_coordinator.py # Subagent lifecycle management
│   │   ├── tool_permissions.py # Tool access control system (421 lines)
│   │   ├── terminal_manager.py # Terminal state management
│   │   ├── global_interrupt.py # Centralized interrupt handling
│   │   └── [15 additional core modules]
│   ├── providers/               # API provider implementations
│   │   ├── __init__.py         # Provider exports
│   │   ├── anthropic_provider.py # Anthropic API provider
│   │   ├── openai_provider.py  # OpenAI API provider
│   │   ├── openrouter_provider.py # OpenRouter API provider
│   │   ├── deepseek_provider.py # DeepSeek API provider
│   │   └── google_provider.py  # Google Gemini API provider
│   ├── tools/                   # Tool integration and execution
│   │   ├── __init__.py         # Tool exports
│   │   └── builtin_tools.py    # Built-in tool definitions (475 lines)
│   ├── utils/                   # Utility functions (7 utility modules)
│   │   ├── __init__.py         # Utility exports
│   │   ├── tool_conversion.py  # Multi-format tool schema conversion
│   │   ├── content_processing.py # Content extraction and cleaning utilities
│   │   ├── diff_display.py     # Terminal diff visualization with colors
│   │   ├── http_client.py      # HTTP client factory and lifecycle management
│   │   ├── retry.py            # Exponential backoff retry logic
│   │   └── tool_parsing.py     # Tool call parsing from LLM responses
│   ├── mcp/                     # MCP protocol implementations
│   │   ├── __init__.py         # MCP module initialization
│   │   └── model_server.py     # MCP model server implementation
│   │   ├── retry.py            # Retry logic utilities
│   │   └── tool_parsing.py     # Tool parsing utilities
│   ├── cli/                     # Command-line interface (future expansion)
│   │   └── __init__.py
│   └── subagents/              # Subagent management (future expansion)
│       └── __init__.py
├── agent.py                     # Main CLI entry point (505 lines)
├── mcp_deepseek_host.py        # Legacy DeepSeek implementation (deprecated)
├── mcp_gemini_host.py          # Legacy Gemini implementation (deprecated)
├── config.py                   # Configuration management with provider-model support
├── subagent.py                 # Subagent subprocess management
└── README.md                   # Project documentation
```

## 🏗️ Provider-Model Architecture

The system uses a sophisticated Provider-Model architecture that separates API integration concerns from model-specific behavior:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amranu/cli-agent](https://github.com/amranu/cli-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
