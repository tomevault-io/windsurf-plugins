---
trigger: always_on
description: This file provides guidance to AI agents when working with code in this repository.
---

# Repository Guidance

This file provides guidance to AI agents when working with code in this repository.
Additional documentation is available in the `docs` folder if needed.

## Essential Commands

### Building and Development
- `cargo check` - Test if the project compiles
- `cargo check --tests` - Test if the tests compile
- `cargo test` - Run all tests
- `cargo fmt --all -- --check` - Check code formatting
- `cargo clippy --all-targets --all-features -- -D warnings` - Run linter

### Testing Specific Components
- `cargo test --package code-assistant` - Test main crate
- `cargo test --package llm` - Test LLM integration
- `cargo test --package web` - Test web functionality

## Architecture Overview

This is a Rust-based tool for AI-assisted code tasks with multiple operational modes.

### Core Structure
- **Workspace Layout**: Multi-crate workspace with 3 main crates:
  - `crates/code_assistant/` - Main application logic
  - `crates/llm/` - LLM provider integrations (Anthropic, OpenAI, Vertex, Ollama, etc.)
  - `crates/web/` - Web-related functionality (Perplexity, web fetches)

### Key Components
- **Agent System** (`src/agent/`): Core AI agent logic with persistence and tool execution
- **Tool System** (`src/tools/`): Extensible tool framework with implementations for file operations, command execution, and web searches
- **UI Framework** (`src/ui/`): Dual interface support:
  - Terminal UI with rustyline
  - GUI using Zed's GPUI framework
- **Session Management** (`src/session/`): Multi-session support with persistence
- **MCP Server** (`src/mcp/`): Model Context Protocol server implementation

### Tool Architecture
- **Core Framework** (`src/tools/core/`): Dynamic tool registry and execution system
- **Tool Implementations** (`src/tools/impls/`): File operations, command execution, search, web fetch
- **Tool Modes**:
  - `native` - Uses LLM provider's native tool calling
  - `xml` - Custom XML-based tool syntax in system messages
  - `caret` - Custom triple-caret-fenced tool syntax in system messages

### LLM Integration
- Multi-provider support: Anthropic, OpenAI, Google Vertex, Ollama, OpenRouter, AI Core
- Recording/playback system for debugging and testing
- Configurable context windows and model selection

## Configuration

### MCP Server Mode
- Integrates with Claude Desktop as MCP server

### Agent Mode
- Supports both terminal, Agent Client Protocol, and GUI interfaces
- State persistence for continuing sessions

## Development Notes

### Testing
- Unit tests distributed across modules
- Integration tests in `src/tests/`
- Mock implementations for testing (`src/tests/mocks.rs`)

### UI Development
- GUI based on Zed's gpui and gpui-component with custom components
- Streaming JSON/XML/Caret processors for real-time updates
- Theme support

### Tool Development
- Implement `DynTool` trait for new tools
- Register in tool registry
- Support both sync and async operations
- Follow existing patterns in `src/tools/impls/`

## UI Communication Architecture

### Communication Channels
There are **two main communication patterns** between components and the UI:

1. **Direct UserInterface trait calls** (primary pattern):
   - Agent calls methods like `begin_llm_request()`, `display()`, `update_tool_status()`, etc.
   - The `display()` method takes a `UIMessage` which can wrap a `UiEvent`
   - Can send any UI event by wrapping it: `UIMessage::UiEvent(event)`
   - Events go into the main UI event queue processed by the first task

2. **Backend thread communication** (session management):
   - Used for session management operations (create, delete, list sessions)
   - Has separate `BackendEvent`/`BackendResponse` types and channels
   - Handled by a second task running concurrently
   - Operations: `LoadSession`, `CreateNewSession`, `ListSessions`, etc.

### Event Queue Architecture
- **Two event queues** running concurrently with separate tasks
- **Task 1**: Processes `UiEvent`s from UserInterface trait calls
- **Task 2**: Handles session management `BackendEvent`s and `BackendResponse`s
- Architecture acknowledged as "messy" and should be cleaned up eventually

### Concurrent Agent System
- **Multiple agents** can run concurrently, one per session
- **Only one agent** is connected to the UI at any time
- **ProxyUI system**: Each session gets a `ProxyUI` instance that only forwards events and method calls to the real UI when that session is "connected"
- **Session states**:
  - **Connected**: Session is actively connected to UI (user clicked on chat item in sidebar)
  - **Active**: Agent loop is currently running in the session (can be active without being connected)
- **Session switching**: User clicks chat items in sidebar to connect/activate different sessions

### Key Implementation Details
- Agent-to-UI communication should use existing `self.ui.display(UIMessage::UiEvent(...))` pattern
- Avoid overcomplicating with new channels or architectures
- Leverage the ProxyUI system for proper session isolation
- Session metadata updates can be sent directly via the existing event system

(Below instructions copied from Zed's `.rules` file)

## GPUI

GPUI is a UI framework which also provides primitives for state and concurrency management.

### Context


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stippi/code-assistant](https://github.com/stippi/code-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
