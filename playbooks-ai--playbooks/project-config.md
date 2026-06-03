---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Playbooks** is an innovative Python framework for building and executing AI agents using "playbooks" – structured workflows defined in natural language (via Markdown-based .pb files) or Python code. This framework represents a significant step toward Software 3.0, where natural language becomes a first-class programming language.

## Core Architecture

### Framework Components

#### 1. Language & Execution
- **Natural Language Programming**: Write workflows in plain English that compile to executable code
- **Hybrid Execution Stack**: Seamless interoperability between Markdown (natural language) and Python execution
- **Compiler-Driven Reliability**: `.pb` files compile to `.pbasm` (assembly) format with semantic static analysis

#### 2. Agent System
- **Multi-Agent Architecture**: Built-in support for distributed agents with communication protocols
- **Agent Types**: Local AI agents, remote AI agents, human agents, MCP agents, system agents
- **Agent Registry**: Dynamic agent discovery and integration capabilities

#### 3. Execution Engine
- **Program Class**: Core execution orchestrator managing playbook lifecycle
- **Event-Driven Architecture**: Reactive programming with triggers and event bus
- **Call Stack Management**: Unified call stack for both natural language and Python execution

### Key Architectural Insights

#### Compilation Pipeline
The compilation flow from `.pb` → `.pbasm` involves:
1. **Loader** (`loader.py`) reads and validates playbook files
2. **Compiler** (`compiler.py`) uses LLM to preprocess content, adding line type codes (QUE, CND, etc.)
3. **AST Generation** (`markdown_to_ast.py`) converts compiled content to executable structure
4. **Program** (`program.py`) orchestrates execution using the compiled AST

#### Agent Threading Model
- Migrated from threading to pure async/await architecture (`AsyncAgentRuntime` in `program.py`)
- Each agent runs as an independent asyncio task
- Message passing between agents is handled through the event bus
- Agent lifecycle managed through `start_agent()` and `stop_agent()` methods

#### Message Flow Architecture
1. **Messages** created with specific types (MessageType enum)
2. **Event Bus** (`event_bus.py`) handles message routing between agents
3. **Agents** subscribe to specific message patterns
4. **Execution State** (`execution_state.py`) maintains context across message handling

#### Playbook Execution Modes
- **Local Playbooks**: Direct execution within the same process
- **Remote Playbooks**: Network-based execution via MCP transport
- **Python Playbooks**: Direct Python function execution with decorator support
- **LLM Playbooks**: Dynamic playbook generation and execution

#### MCP Integration
- **Transport Layer** (`transport/mcp_transport.py`): Handles WebSocket/SSE connections
- **MCP Agents** (`mcp_agent.py`): Bridge between MCP servers and playbook system
- **Protocol** (`transport/protocol.py`): Defines message format and handshake

## Key Features

### 1. Compilation System
- **Semantic Static Analysis**: Infers intent, adds annotations (QUE, CND), variables/types
- **Intermediate Representation**: `.pb` → `.pbasm` compilation for reliable execution
- **Runtime Optimization**: Reduces LLM hallucinations through structured guidance

### 2. Agent Communication
- **MCP (Model Context Protocol)**: Standard for agent-to-agent communication
- **Transport Layer**: WebSocket, SSE, and other transport protocols
- **Message Routing**: Intelligent message routing between agents

### 3. Development Tools
- **CLI Interface**: Command-line tools for running and compiling playbooks
- **VSCode Extension**: Debugging and development support
- **Rich Console Output**: Enhanced terminal experience with rich formatting

## Project Structure

```
playbooks/
├── src/playbooks/
│   ├── agents/           # Agent implementations and registry
│   ├── applications/     # Application interfaces (CLI, web, Streamlit)
│   ├── common/          # Shared utilities and logging
│   ├── debug/           # Debugging infrastructure
│   ├── meetings/        # Meeting management system
│   ├── playbook/        # Playbook implementations (local, remote, etc.)
│   ├── prompts/         # LLM prompts and templates
│   ├── transport/       # Communication protocols
│   └── utils/           # Utility functions and helpers
├── tests/               # Test suite
│   ├── data/           # Test playbooks and examples
│   └── unit/           # Unit tests
└── docs/               # Documentation
```

## Core Classes & Modules

### Essential Components

#### Main Entry Points
- `main.py`: Primary Playbooks class orchestrating the entire framework
- `cli.py`: Command-line interface for running and compiling playbooks
- `compiler.py`: Compiles `.pb` files to `.pbasm` intermediate representation

#### Execution Engine
- `program.py`: Core execution orchestrator managing playbook lifecycle
- `execution_state.py`: Manages execution state and context
- `call_stack.py`: Unified call stack for hybrid execution
- `event_bus.py`: Event-driven architecture for reactive programming

#### Agent System

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [playbooks-ai/playbooks](https://github.com/playbooks-ai/playbooks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
