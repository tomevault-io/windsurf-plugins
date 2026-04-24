---
trigger: always_on
description: This file provides guidance for Coding Agents working in this repository.
---

# Sagents AI Library

This file provides guidance for Coding Agents working in this repository.

## Project Overview

The Sagents library is a framework for integrating more advanced conversational AI Agents into Elixir applications.

It provides `mix task` generators for integrating more easily into an Elixir application. It provides advanced middleware for composably adding features to an agent..

It draws initial inspiration from LangChain Deep Agents library, but follows it's own path that aims for integration with Elixir ecosystem.

## Common Development Commands

### Setup and Dependencies
```bash
# Install dependencies
mix deps.get

# Set up environment variables for API keys
cp .envrc_template .envrc
# Edit .envrc with API keys, then:
source .envrc
```

Never read the `.envrc` file as it contains developer secrets.

### Testing
```bash
# Run unit tests only (makes no external API calls)
mix test

# Run specific live API tests (billable and should be confirmed first)
mix test --include live_call
mix test --include live_anthropic

# Run a single test file
mix test test/path/to/test_file.exs

# Run a specific test by line number
mix test test/path/to/test_file.exs:42
```

### Code Quality

Always run this before committing:

```bash
# Performs a set of pre-commit steps of compile checks, formatting, running test, etc.
mix precommit
```

## High-Level Architecture

### Core Components

1. **Chat Models** (`lib/chat_models/`)
   - `ChatModel` behavior defines the interface for all LLM implementations
   - Each provider (OpenAI, Anthropic, etc.) implements this behavior
   - Supports streaming, function calling, and multi-modal inputs

2. **Chains** (`lib/chains/`)
   - `LLMChain`: Primary abstraction for core orchestration of conversations and other LLM workflows
   - `DataExtractionChain`: Structured data extraction from text
   - `RoutingChain`: Dynamic routing based on input
   - Chains compose multiple operations and maintain conversation state

3. **Messages** (`lib/message.ex` and `lib/message/`)
   - `Message`: Core structure with roles (system, user, assistant, tool)
   - `ContentPart`: Handles multi-modal content (text, images, files)
   - `ToolCall` and `ToolResult`: Function invocation and results

4. **Functions** (`lib/function.ex`)
   - Integrates custom Elixir functions with LLMs
   - JSON Schema-based parameter validation
   - Context-aware execution with async support

### Key Patterns

- **Behavior-based design**: All chat models implement the `ChatModel` behavior
- **Ecto schemas**: Used for data validation and type casting throughout
- **Streaming support**: Built-in streaming capabilities for real-time responses
- **Error handling**: Consistent error tuples `{:ok, result}` / `{:error, reason}`

### Adding New Features

When adding a new LLM provider:
1. Create a new module in `lib/chat_models/`
2. Implement the `ChatModel` behavior
3. Add corresponding tests in `test/chat_models/`
4. Update documentation with supported features

When adding new chain types:
1. Create in `lib/chains/` following existing patterns
2. Use `Ecto.Schema` for configuration
3. Implement `run/2` function for execution
4. Add comprehensive tests including async scenarios

## Testing Guidelines

- Tests mirror the source structure (e.g., `lib/chains/llm_chain.ex` → `test/chains/llm_chain_test.exs`)
- Use `@tag :live_call` for tests requiring actual API calls
- Mock external dependencies using `Mimic` for unit tests
- Always test both sync and async execution paths when applicable

## Important Notes

- **API Keys**: Never commit API keys. Use environment variables via `.envrc`
- **Live Tests**: Be cautious with live tests as they incur API costs
- **Multi-modal**: When working with messages, use `ContentPart` structures
- **Callbacks**: There are two callback systems that get combined into a single list of handler maps:
  1. **LangChain callbacks** — keys like `:on_llm_token_usage`, `:on_message_processed`, etc. These are added to the LLMChain via `add_callback/2` and fired by `LLMChain.run/2`. See `LangChain.Chains.ChainCallbacks` for valid keys.
  2. **Sagents-specific callbacks** — currently just `:on_after_middleware`, fired manually by `Agent.execute/3` after `before_model` hooks complete. Not a LangChain key.

  Both types live in the same list of maps passed via `callbacks: [map1, map2, ...]`. AgentServer builds this list by combining PubSub broadcast callbacks (`build_pubsub_callbacks/1`) with middleware callbacks (`Middleware.collect_callbacks/1`). All maps fire in fan-out: if two maps have the same key, both handlers fire. Middleware can declare LangChain callbacks via the `callbacks/1` behaviour callback.


## Project guidelines

- Use `mix precommit` alias when you are done with all changes and fix any pending issues
- Use the already included and available `:req` (`Req`) library for HTTP requests, **avoid** `:httpoison`, `:tesla`, and `:httpc`. Req is included by default and is the preferred HTTP client for Phoenix apps

### Phoenix v1.8 guidelines

- **Always** begin your LiveView templates with `<Layouts.app flash={@flash} ...>` which wraps all inner content

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sagents-ai/sagents](https://github.com/sagents-ai/sagents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
