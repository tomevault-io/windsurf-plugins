---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Dartantic is an agentic AI framework for Dart that provides easy integration with multiple AI providers (OpenAI, OpenAI Responses API, Google, Anthropic, Mistral, Cohere, Ollama, OpenRouter, xAI / Grok via OpenAI-compatible chat completions, xAI Responses / Grok via the Responses API). The optional `packages/dartantic_firebase_ai/` package adds Gemini through Firebase AI Logic for Flutter. It features streaming output, typed responses, tool calling, embeddings, and MCP (Model Context Protocol) support.

The project is organized as a monorepo with multiple packages:
- `packages/dartantic_interface/` - Core interfaces and types shared across all Dartantic packages
- `packages/dartantic_ai/` - Main implementation with provider integrations (primary development focus)
- `packages/dartantic_firebase_ai/` - Flutter-only Firebase AI Logic provider (`firebase-google` / `firebase-vertex` on `Agent.providerFactories`)
- `packages/dartantic_chat/` - Flutter chat UI widgets for AI applications (fork of flutter/ai toolkit)
- `samples/dartantic_cli/` - Command-line interface for the Dartantic framework
- `samples/chatarang/` - Interactive command-line chat application with tool support

## Documentation

- **External Docs**: Full documentation at [docs.dartantic.ai](https://docs.dartantic.ai)
- **Wiki Documentation**: The `wiki/` folder contains comprehensive architecture documentation. See `wiki/Home.md` for the complete index of design documents, specifications, and implementation guides.
- **Design documents should NOT include code implementations** - Specifications in the `wiki/` folder should describe algorithms, data flow, and architecture without including actual code, as code in documentation immediately goes stale. Implementation details belong in the code itself, not in design docs.

## Development Commands

### Building and Testing
```bash
# Run all tests in the dartantic_ai package
cd packages/dartantic_ai && dart test

# Run a specific test file
cd packages/dartantic_ai && dart test test/specific_test.dart

# Run tests matching a name pattern
cd packages/dartantic_ai && dart test -n "pattern"

# Run a single test by name
cd packages/dartantic_ai && dart test -n "test name"

# Analyze code for issues
cd packages/dartantic_ai && dart analyze

# Format code
cd packages/dartantic_ai && dart format .

# Check formatting without making changes
cd packages/dartantic_ai && dart format --set-exit-if-changed .
```

### Running Examples
```bash
# Run example files (from dartantic_ai package)
cd packages/dartantic_ai && dart run example/bin/single_turn_chat.dart
cd packages/dartantic_ai && dart run example/bin/typed_output.dart
cd packages/dartantic_ai && dart run example/bin/tool_calling.dart

# Run dartantic_chat Flutter examples (requires API key)
cd packages/dartantic_chat/example && flutter run --dart-define=GEMINI_API_KEY=$GEMINI_API_KEY
```

### Debugging
```bash
# Enable detailed logging via environment variable
DARTANTIC_LOG_LEVEL=FINE dart run example/bin/single_turn_chat.dart

# Log levels: SEVERE, WARNING, INFO, FINE (most verbose)
DARTANTIC_LOG_LEVEL=INFO dart test test/specific_test.dart
```

### Package Management
```bash
# Get dependencies
cd packages/dartantic_ai && dart pub get

# Upgrade dependencies
cd packages/dartantic_ai && dart pub upgrade
```

### Dartantic CLI Development
```bash
# Run the CLI (from samples/dartantic_cli directory)
cd samples/dartantic_cli && dart run bin/dartantic.dart -p "Hello"

# Run CLI tests
cd samples/dartantic_cli && dart test

# Run a single CLI test
cd samples/dartantic_cli && dart test test/cli_test.dart

# Run all CLI example scripts
cd samples/dartantic_cli && bash example/run_all.sh

# Run a single example
cd samples/dartantic_cli && bash example/basic/simple_chat.sh
```

## Architecture

### Six-Layer Architecture

Dartantic uses a six-layer architecture with clear separation of concerns:

1. **API Layer** (`lib/src/agent/agent.dart`)
   - Thin coordination layer - main user-facing interface
   - Model string parsing and provider selection
   - Conversation state management
   - Public API contracts

2. **Orchestration Layer** (`lib/src/agent/orchestrators/`)
   - Complex workflow management (streaming, tool execution, typed output)
   - `DefaultStreamingOrchestrator` - Standard chat workflows
   - `TypedOutputStreamingOrchestrator` - Structured JSON output
   - `StreamingState` - Encapsulated mutable state per request
   - `ToolExecutor` - Centralized tool execution with error handling

3. **Provider Abstraction Layer** (`packages/dartantic_interface/`)
   - Clean contracts independent of implementation
   - Provider interface with capability declarations
   - ChatModel and EmbeddingsModel interfaces
   - Core types re-exported from `genai_primitives` (ChatMessage, Part types, ToolDefinition)
   - Schema construction via `json_schema_builder` (use `S.*` builder methods)

4. **Provider Implementation Layer** (`lib/src/providers/`, `lib/src/chat_models/`, `lib/src/embeddings_models/`)
   - Provider-specific implementations isolated

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [csells/dartantic](https://github.com/csells/dartantic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
