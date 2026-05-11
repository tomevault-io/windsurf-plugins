---
trigger: always_on
description: This file provides guidance to AI development assistants when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI development assistants when working with code in this repository.

**Supported AI Assistants:**
- IBM Bob
- Claude Code
- GitHub Copilot
- Cursor AI
- Windsurf
- Gemini CLI
- Any AI assistant with codebase context awareness

## Project Overview

qiskit-mcp-servers is a collection of **Model Context Protocol (MCP)** servers that provide AI assistants, LLMs, and agents with seamless access to IBM Quantum services and Qiskit libraries for quantum computing development and research.

### Core Purpose
- Enable AI systems to interact with quantum computing resources through Qiskit
- Provide production-ready MCP servers for quantum computing workflows
- Connect AI assistants to real quantum hardware automatically
- Simplify quantum circuit execution and job management
- Provide intelligent quantum code completion and assistance
- Enable AI-powered quantum circuit optimization and transpilation

### Key Technologies
- **Protocol**: Model Context Protocol (MCP)
- **Language**: Python 3.10+ (3.11+ recommended)
- **Framework**: FastMCP (async-first MCP framework)
- **Package Manager**: uv (modern Python package manager with workspace support)
- **Testing**: pytest with async support, 65%+ coverage
- **Code Quality**: ruff (formatting + linting), mypy (type checking)
- **Build System**: hatchling with pyproject.toml

## Architecture

### Repository Structure

This is a **monorepo** using uv workspace containing six independent MCP servers:

```
qiskit-mcp-servers/
├── qiskit-mcp-server/                       # Core Qiskit transpilation
├── qiskit-code-assistant-mcp-server/        # AI code completion
├── qiskit-docs-mcp-server/                  # Documentation retrieval
├── qiskit-ibm-runtime-mcp-server/           # IBM Quantum cloud services
├── qiskit-ibm-transpiler-mcp-server/        # AI-powered transpilation
├── qiskit-gym-mcp-server/                   # RL-based circuit synthesis
├── .claude/skills/                          # Claude Code skills
├── .github/                                 # GitHub templates and workflows
├── pyproject.toml                           # Workspace configuration & meta-package
├── mypy.ini                                 # Shared mypy configuration
├── ruff.toml                                # Shared ruff configuration
├── README.md                                # Main repository documentation
├── PUBLISHING.md                            # PyPI publishing guide
├── CONTRIBUTING.md                          # Contribution guidelines
├── CODE_OF_CONDUCT.md                       # Community guidelines
└── LICENSE                                  # Apache 2.0 license
```

Each server is:
- **Independent**: Can be installed and run separately
- **Self-contained**: Has its own dependencies and tests
- **Publishable**: Separate PyPI packages
- **Consistent**: Follows unified design principles

### Workspace Configuration

The root `pyproject.toml` defines a uv workspace:

```toml
[tool.uv.workspace]
members = [
    "qiskit-mcp-server",
    "qiskit-code-assistant-mcp-server",
    "qiskit-docs-mcp-server",
    "qiskit-ibm-runtime-mcp-server",
    "qiskit-ibm-transpiler-mcp-server",
    "qiskit-gym-mcp-server",
]
```

The root package is also a **meta-package** that installs all servers:
```bash
pip install qiskit-mcp-servers  # Installs all six servers
```

### Component Structure

Each MCP server follows this standard structure:

```
<server-name>/
├── src/
│   └── <package_name>/
│       ├── __init__.py          # Main entry point
│       ├── server.py            # FastMCP server definition
│       ├── <core>.py            # Core functionality (async)
│       └── utils.py             # Utilities (optional)
├── tests/
│   ├── conftest.py              # Test fixtures
│   ├── unit/                    # Unit tests (optional subdirectory)
│   ├── integration/             # Integration tests (optional subdirectory)
│   └── test_*.py                # Test files
├── examples/
│   ├── README.md                # MCP server example documentation
│   ├── langchain_agent.ipynb    # Interactive tutorial with step-by-step examples
│   └── langchain_agent.py       # Command-line agent with multiple LLM provider support
├── pyproject.toml               # Project metadata & dependencies
├── server.json                  # MCP Registry metadata (for publishing)
├── pytest.ini                   # pytest configuration (optional)
├── LICENSE                      # Apache 2.0 license (copy from root)
├── README.md                    # Server-specific documentation
├── .env.example                 # Environment variable template (optional)
└── run_tests.sh                 # Test execution script
```

## Key Components

### 1. Qiskit MCP Server

**Purpose**: Core quantum circuit transpilation using Qiskit pass managers

**Directory**: [`qiskit-mcp-server/`](qiskit-mcp-server/)

**Core Files**:
- `server.py`: FastMCP server with tool/resource definitions
- `transpiler.py`: Qiskit transpilation functions (async)
- `circuit_serialization.py`: QASM3/QPY conversion utilities
- `utils.py`: Basis gate presets and coupling map topologies

**Tools Provided**:
| Tool | Description |
|------|-------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Qiskit/mcp-servers](https://github.com/Qiskit/mcp-servers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
