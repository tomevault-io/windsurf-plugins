---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Aiex is a sophisticated distributed AI-powered Elixir coding assistant currently in Phase 6 of a 9-phase development roadmap. The project has built extensive distributed infrastructure using pure OTP primitives and is now implementing the core AI assistant application logic that provides actual coding assistance capabilities. Aiex leverages Elixir's strengths in concurrency, fault tolerance, and distributed computing to create a production-ready AI coding assistant.

## Current State (End of Phase 5)

Aiex has completed 5 major phases and now has sophisticated distributed infrastructure in place:

- **Distributed OTP Architecture**: Complete with pg process group coordination, Mnesia persistence, and event sourcing
- **Multi-LLM Integration**: 4 providers (OpenAI, Anthropic, Ollama, LM Studio) with intelligent coordination, circuit breakers, and health monitoring
- **Advanced Context Management**: Semantic chunking, context compression, and distributed synchronization across nodes
- **Sophisticated TUI**: Rust-based terminal interface with real-time OTP communication via TCP/MessagePack
- **Event Sourcing System**: Complete audit trail with distributed event bus using pg module
- **Multiple Interfaces**: CLI, TUI, Mix tasks with unified business logic through InterfaceGateway
- **Security & Sandboxing**: Path validation, audit logging, and secure file operations
- **Rich Terminal Features**: Syntax highlighting, real-time context awareness, and multi-panel layouts

**What's Missing**: The actual AI assistant logic that provides coding assistance (analysis, generation, explanation, refactoring).

## Common Development Commands

### Build and Dependencies
- `mix deps.get` - Install dependencies
- `mix compile` - Compile the project
- `make dev` - Quick development start (runs `iex -S mix`)
- `make release` - Build production release
- `make test` - Run test suite

### Testing
- `mix test` - Run all tests
- `mix test test/aiex_test.exs` - Run specific test file
- `mix test test/aiex_test.exs:5` - Run test at specific line
- `mix test --only integration` - Run integration tests only
- `mix test --exclude integration` - Skip integration tests

### Code Quality
- `mix format` - Format code according to Elixir standards
- `mix format --check-formatted` - Check if code is properly formatted
- `mix dialyzer` - Run static analysis (if configured)

### Interactive Development
- `iex -S mix` - Start interactive Elixir shell with project loaded
- `aiex start-iex` - Start OTP application with interactive shell
- `./aiex help` - Test escript CLI interface

### TUI Development
- `cd tui && cargo run` - Run Rust TUI client
- `cd tui && cargo run -- --debug` - Run TUI with debug logging
- `cd tui && cargo test` - Run TUI unit tests
- `cd tui && cargo fmt` - Format Rust code
- `cd tui && cargo clippy` - Run Rust linting

### AI Assistant Testing
- `mix ai.explain lib/aiex.ex` - Test AI explanation functionality
- `mix ai.gen.module Calculator "arithmetic operations"` - Test AI module generation
- `./aiex cli analyze lib/` - Test CLI analysis command
- `./aiex cli create module TestModule "description"` - Test CLI creation

### Performance and Monitoring
- `RUST_LOG=debug ./aiex start-iex` - Enable debug logging
- `:observer.start()` - Launch Erlang Observer for monitoring (in IEx)
- `:recon.proc_count(:memory, 10)` - Show top memory processes (in IEx)

## Working Method

We use a structured approach combining todo tracking with documentation for implementing AI assistant features.

### Current Phase 6 Workflow:

1. **Todo Tracking**: Use TodoWrite/TodoRead tools to track AI assistant implementation progress
2. **Plan Before Implementation**: Always create and review plans before coding (NO EXCEPTIONS!)
3. **Test-Driven Development**: Write comprehensive tests for all AI functionality
4. **Integration Testing**: Ensure AI engines work with existing distributed infrastructure
5. **Documentation**: Document AI assistant patterns and integration points

### Feature Implementation Workflow:

1. **Plan Phase**: Collaborate on design, save plan in `/notes/features/<number>-<name>.md` under `## Plan` heading
2. **Implementation Phase**: Store notes, findings, issues in `/notes/features/<number>-<name>.md` under `## Log` heading  
3. **Testing & Finalization**: Document final design in `/notes/features/<number>-<name>.md` under `## Conclusion` heading

### Bug Fix Workflow:

1. **Issue Documentation**: Document in `/notes/fixes/<number>-<name>.md` under `## Issue` heading
2. **Fix Implementation**: Store technical details in `/notes/fixes/<number>-<name>.md` under `## Fix` heading
3. **Resolution Summary**: Document learnings in `/notes/fixes/<number>-<name>.md` under `## Conclusion` heading

### AI Assistant Development Rules:

- **Always plan first**: Refuse to implement until plan is created and reviewed
- **Use existing infrastructure**: Leverage LLM coordination, context management, event sourcing
- **Follow OTP patterns**: Maintain supervision tree and distributed architecture

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pcharbon70) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
