---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This repository contains three integrated Ruby gems that work together to create collaborative AI agent systems:

- **SwarmSDK** (`lib/swarm_sdk/`) - Core SDK for building multi-agent AI systems using RubyLLM
- **SwarmMemory** (`lib/swarm_memory/`) - Persistent memory system with semantic search for agents
- **SwarmCLI** (`lib/swarm_cli/`) - Modern command-line interface using TTY toolkit components

### SwarmSDK

SwarmSDK is a single-process, lightweight framework for building collaborative AI agent systems. Key features:

- **Single Process**: All agents run in the same Ruby process using RubyLLM
- **Agent Delegation**: Agents can delegate tasks to specialized agents
- **Tool System**: Rich set of built-in tools (Read, Write, Edit, Bash, etc.)
- **Ruby DSL**: Clean, intuitive API for defining agent swarms
- **Event System**: Comprehensive event emission for monitoring and debugging
- **State Management**: Snapshot/restore for session persistence
- **Async Support**: Built on Async gem for efficient concurrent execution
- **MCP Integration**: Connect to external MCP servers via RubyLLM::MCP

### SwarmMemory

SwarmMemory provides hierarchical persistent memory with semantic search capabilities:

- **Semantic Search**: Fast ONNX-based embeddings using Informers gem
- **Memory Tools**: MemoryWrite, MemoryRead, MemoryEdit, MemoryDelete, MemoryGrep, MemoryGlob
- **SDK Integration**: Seamlessly integrates with SwarmSDK through plugin system
- **Frontmatter Support**: Extract metadata from memory entries
- **Storage Abstraction**: Pluggable storage backends
- **Defragmentation**: Optimize memory storage over time

### SwarmCLI

SwarmCLI provides a modern, user-friendly command-line interface:

- **Dual-Mode Support**: Interactive REPL and non-interactive automation
- **TTY Toolkit**: Rich terminal UI with Pastel styling, TTY::Spinner, TTY::Markdown
- **JSON Logging**: Structured logs for automation and scripting
- **Reline Integration**: Advanced line editing with history and completion
- **Configuration Management**: YAML-based swarm definitions
- **Memory Commands**: Full memory system integration (if swarm_memory installed)

## Development Commands

### Testing
NEVER RUN ALL TESTS WITH `bundle exec rake test`. Run tests for each component separately.

Each component has its own test suite:

```bash
# Run specific component tests
bundle exec rake swarm_sdk:test      # SwarmSDK tests
bundle exec rake swarm_memory:test   # SwarmMemory tests
bundle exec rake swarm_cli:test      # SwarmCLI tests
```

**Important**: Tests should not generate any output to stdout or stderr. When writing tests:
- Capture or suppress all stdout/stderr output from tested methods
- Use `capture_io` or `capture_subprocess_io` for Minitest
- Redirect output streams to `StringIO` or `/dev/null` when necessary
- Mock or stub methods that produce console output
- Ensure clean test output for better CI/CD integration

Example:
```ruby
def test_command_with_output
  output, err = capture_io do
    # Code that produces output
  end
  # Test assertions here
end
```

### Linting

```bash
bundle exec rubocop -A       # Run RuboCop linter to auto fix problems
```

## Architecture

### SwarmSDK Architecture (`lib/swarm_sdk/`)

**Core Components:**

- **Swarm** - Main orchestrator managing multiple agents with shared rate limiting
- **Agent::Definition** - Agent configuration and validation
- **Agent::Chat** - RubyLLM chat wrapper with tool execution and delegation
- **AgentInitializer** - Complex 5-pass agent setup with tool and MCP configuration
- **ToolConfigurator** - Tool creation, permissions, and delegation tool generation
- **McpConfigurator** - MCP client management and configuration
- **NodeOrchestrator** - Multi-node workflow execution with dependencies

**Key Subsystems:**

- **Events**: LogStream + LogCollector for comprehensive event emission
- **Hooks**: Pre/post tool execution, swarm lifecycle hooks
- **State**: Snapshot/restore for session persistence
- **Tools**: 25+ built-in tools (Read, Write, Edit, Bash, Grep, Glob, etc.)
- **Plugins**: Extensible plugin system (SwarmMemory integrates via plugins)

**User-Facing APIs:**

```ruby
# Ruby DSL (Recommended)
swarm = SwarmSDK.build do
  name "Development Team"
  lead :backend

  agent :backend do
    model "claude-sonnet-4"
    description "Backend developer"
    prompt "You build APIs and databases"
    tools :Read, :Edit, :Bash
    delegates_to :database
  end
end

result = swarm.execute("Build authentication system")

# YAML String API
yaml = File.read("swarm.yml")
swarm = SwarmSDK.load(yaml, base_dir: "/path/to/project")

# YAML File API (Convenience)
swarm = SwarmSDK.load_file("swarm.yml")
```

### SwarmMemory Architecture (`lib/swarm_memory/`)

**Core Components:**

- **Core::Storage** - Hierarchical file-based storage with frontmatter
- **Core::Index** - FAISS-based semantic search index
- **Core::Embedder** - ONNX embeddings via Informers gem
- **Tools::Memory*** - Memory manipulation tools (Write, Read, Edit, Delete, etc.)
- **Integration::SDKPlugin** - SwarmSDK plugin for seamless integration


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [parruda/swarm](https://github.com/parruda/swarm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
