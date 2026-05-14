---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## ⚠️ CRITICAL DEVELOPMENT RULES

### 🚫 Git Commit Policy
**NEVER commit changes unless explicitly told by the user.**
- Always ask for permission before running `git commit`
- Only commit when user specifically requests it
- Exception: Only commit automatically when user says "commit this" or similar explicit instruction

### 📦 Version Management Policy
**Always increment PyPI version by patch (0.0.1) unless instructed otherwise.**
- Default increment: `0.2.0` → `0.2.1` → `0.2.2`
- Only use minor/major increments when user explicitly requests
- Always update both `pyproject.toml` and `src/mcp_kicad_sch_api/__init__.py`

### 📋 Feature Development Policy
**Always create a PRD (Product Requirements Document) before implementing features.**
- Write a detailed PRD markdown file in the project root
- Include: Problem statement, proposed solution, technical approach, testing plan
- **ASK USER FOR FEEDBACK** on the PRD before proceeding with implementation
- Do not start coding until user approves the PRD

#### PRD Template:
```markdown
# PRD: [Feature Name]

## Problem Statement
[What problem are we solving?]

## Proposed Solution
[High-level approach]

## Technical Implementation
- [Technical details]
- [API changes]
- [Testing approach]

## Success Criteria
[How do we know it works?]
```

### 🔄 Required Workflow
1. **User requests feature** → Write PRD → **Ask for user approval**
2. **User approves PRD** → Implement feature → Run tests
3. **Implementation complete** → **Ask user if they want to commit**
4. **User says commit** → Commit with appropriate message
5. **User requests release** → Increment patch version → Publish to PyPI

## Project Overview

mcp-kicad-sch-api is a **Model Context Protocol (MCP) server** that provides KiCAD schematic manipulation tools for AI agents. It acts as a bridge between AI systems and KiCAD schematic files, enabling automated circuit design and analysis.

## Architecture

```
mcp-kicad-sch-api/
├── src/mcp_kicad_sch_api/           # MCP Server implementation
│   ├── __init__.py                  # Package initialization & CLI
│   ├── __main__.py                  # Entry point for server execution
│   └── server.py                    # Core MCP server with tool handlers
├── tests/                           # Comprehensive test suite
│   ├── test_server.py              # Basic server tests
│   └── test_mcp_server_integration.py # Integration tests with real KiCAD API
├── submodules/                      # Git submodules
│   └── kicad-sch-api/              # Core KiCAD manipulation library
├── pyproject.toml                   # Package configuration
└── README.md                        # User documentation
```

## Key Commands

### Development Environment Setup
```bash
# Install in development mode
pip install -e .

# Install with dependencies from submodule
pip install -e "submodules/kicad-sch-api"

# Install MCP SDK
pip install mcp
```

### Testing Commands
```bash
# Run all tests
pytest tests/ -v

# Run integration tests (requires kicad-sch-api)
pytest tests/test_mcp_server_integration.py -v

# Run specific test method
pytest tests/test_mcp_server_integration.py::TestMCPServerIntegration::test_add_component_workflow -v
```

### MCP Server Commands
```bash
# Run MCP server directly
python -m mcp_kicad_sch_api

# Run MCP server with verbose logging
python -m mcp_kicad_sch_api -vv

# Test server starts without errors
python -c "import sys; sys.path.insert(0, 'src'); from mcp_kicad_sch_api.server import main; print('✅ Server imports successfully')"
```

### Package Management
```bash
# Build package
python -m build

# Upload to PyPI
python -m twine upload dist/mcp_kicad_sch_api-*

# Install latest from PyPI
pip install --upgrade mcp-kicad-sch-api
```

## Core MCP Server API

The server provides these MCP tools for AI agents:

```python
# Available MCP Tools:
- create_schematic(name: str) -> Creates new KiCAD schematic
- load_schematic(file_path: str) -> Loads existing schematic
- save_schematic(file_path?: str) -> Saves current schematic
- add_component(lib_id: str, reference: str, value: str, position: [x,y], footprint?: str) -> Adds component
- add_wire(start_pos: [x,y], end_pos: [x,y]) -> Connects two points
- add_label(text: str, position: [x,y], rotation?: float, size?: float) -> Adds text label
- add_hierarchical_label(text: str, position: [x,y], shape?: str) -> Adds interface label
- add_junction(position: [x,y], diameter?: float) -> Adds connection point
- search_components(query: str, library?: str, limit?: int) -> Searches component libraries
- list_components() -> Lists all components in schematic
- get_schematic_info() -> Gets schematic statistics and info
```

## Testing Strategy & MCP Integration

**CRITICAL**: This project depends on the `kicad-sch-api` library for actual KiCAD manipulation. Always verify MCP tool calls work correctly with real KiCAD files.

### MCP Server Testing
```bash
# Test MCP server functionality
pytest tests/test_mcp_server_integration.py -v

# Test specific workflows
pytest tests/test_mcp_server_integration.py::test_add_component_workflow -v

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [circuit-synth/mcp-kicad-sch-api](https://github.com/circuit-synth/mcp-kicad-sch-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
