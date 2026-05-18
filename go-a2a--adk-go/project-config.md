---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Your Role

You are a Go and Python language developer who provides expert-level insights and solutions, focusing on code translation between these two languages.

Your responses should include best practices, and explanations of underlying concepts.

Remember:

- Maintain the original functionality and behavior of the Python code.
- Follow Go best practices and idiomatic patterns.
- Do not include the entire Go code in your response, only save it to the specified file.
- If you encounter any insurmountable issues during conversion, explain them clearly in the conversion summary.

## Build/Lint/Test Commands

- Build: `go build ./...`
- Lint: `go vet ./...`
- Test all: `go test ./...`
- Test single: `go test ./path/to/package -run TestName`
- Coverage: `go test -cover ./...`

## Architecture Overview

@include docs/llms-full.txt

This is the Agent Development Kit (ADK) for Go - a code-first toolkit for building, evaluating, and deploying sophisticated AI agents. The architecture follows a hierarchical, event-driven design with strong type safety and extensibility.

### Core Components

#### **Agent System** (`agent/`)
Hierarchical agent architecture with four core agent types:

- **LLMAgent**: Full-featured agents powered by language models with tools, instructions, callbacks, planners, and code execution
- **SequentialAgent**: Executes sub-agents one after another, supports live mode with `taskCompleted()` flow control
- **ParallelAgent**: Runs sub-agents concurrently in isolated branches, merges event streams
- **LoopAgent**: Repeatedly executes sub-agents until escalation or max iterations

Key patterns:
- All agents embed `types.BaseAgent` for common functionality
- Event streaming via `iter.Seq2[*Event, error]` iterators
- Rich `InvocationContext` tracks execution state, session, and hierarchy
- Before/after callbacks for customizing behavior

#### **Type System** (`types/`)
Core interfaces and contracts that all components follow:

- **Agent Interface**: Defines execution, hierarchy navigation, and lifecycle methods
- **Model Interface**: Unified LLM abstraction for content generation and streaming
- **Tool/Toolset**: Extensible tool system with function declarations
- **Session/State**: Conversation and state management abstractions
- **Event System**: Comprehensive event model with actions, deltas, and metadata
- **Python Interop** (`py/`): Go implementations of Python patterns (asyncio, sets)

#### **Model Layer** (`model/`)
Multi-provider LLM integration using `google.golang.org/genai` as the unified interface:

- **Google Gemini**: Direct integration with full streaming and live connection support
- **Anthropic Claude**: Support for direct API, Vertex AI, and AWS Bedrock deployments
- **Registry Pattern**: Pattern-based model resolution with caching
- **Factory Pattern**: Simple model creation with API key injection
- **Connection Management**: Stateful connections for live interactions (Gemini only)

#### **Tool Ecosystem** (`tool/`)
Sophisticated tool framework for external integrations:

- **Base Tools**: Agent tools, function tools, auth tools, code execution tools
- **Tool Context**: Rich context with session, state, artifacts, and auth management
- **Automatic Function Calling**: Reflection-based function declaration generation
- **Authentication Integration**: Multi-scheme auth support (OAuth2, API Key, Basic, Bearer)

#### **Flow Management** (`flow/`)
Pipeline architecture for LLM interaction processing:

- **LLMFlow**: Base flow with request/response processors
- **SingleFlow**: No agent transfers, simple tool calling
- **AutoFlow**: Full agent transfer support (parent/peer)
- **Processors Pipeline**: 
  - Request: Basic, Auth, Instructions, Identity, Content, CodeExecution, AgentTransfer
  - Response: CodeExecution processor for output handling

#### **Code Executors** (`codeexecutor/`)
Secure code execution with multiple backends:

- **BuiltIn**: Uses model's native code execution (Gemini 2.0+)
- **Local**: Direct host execution (requires explicit opt-in)
- **Container**: Docker-based sandboxing with resource limits
- **Features**: Stateful execution, retry logic, language detection, file I/O

#### **Memory Systems** (`memory/`)
Long-term knowledge storage and retrieval:

- **MemoryService Interface**: Add sessions, search memories
- **InMemory**: Simple keyword-based search
- **Vertex AI RAG**: Vector-based retrieval (placeholder)

#### **Session Management** (`session/`)
Stateful conversation tracking:

- **Session**: Tracks events, state, and metadata
- **SessionService**: CRUD operations, event management
- **State Management**: Three-tier state (app, user, temp) with delta tracking
- **InMemoryService**: Reference implementation with thread safety

#### **Planner System** (`planner/`)
Strategic planning for agent execution:

- **BuiltInPlanner**: Leverages model's native thinking (Claude)
- **PlanReActPlanner**: Structured planning/reasoning/action framework

### Key Architectural Patterns

1. **Event-Driven Streaming**: All operations use `iter.Seq2[*Event, error]` for real-time processing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [go-a2a/adk-go](https://github.com/go-a2a/adk-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
