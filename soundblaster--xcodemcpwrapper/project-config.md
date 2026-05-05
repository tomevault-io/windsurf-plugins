---
trigger: always_on
description: This project provides a wrapper solution that enables external AI agents (Cursor, Claude CLI, Codex) to connect to Xcode via the Model Context Protocol (MCP). Xcode 26.3+ includes an MCP bridge (`xcrun mcpbridge`) that exposes Xcode's internal capabilities to MCP clients, but it has a protocol compatibility issue that prevents it from working with strict MCP spec followers like Cursor.
---

# Xcode MCP Wrapper

## Project Overview

This project provides a wrapper solution that enables external AI agents (Cursor, Claude CLI, Codex) to connect to Xcode via the Model Context Protocol (MCP). Xcode 26.3+ includes an MCP bridge (`xcrun mcpbridge`) that exposes Xcode's internal capabilities to MCP clients, but it has a protocol compatibility issue that prevents it from working with strict MCP spec followers like Cursor.

### The Problem

Xcode's `mcpbridge` returns tool responses in the `content` field but omits the required `structuredContent` field when a tool declares an `outputSchema`. According to the MCP specification, when `outputSchema` is declared, responses **must** include `structuredContent`. Claude Code and Codex CLI work because they have special handling for Apple's responses; Cursor strictly follows the spec and rejects non-compliant responses.

### The Solution

A Python wrapper (`xcodemcpwrapper`) that intercepts responses from `xcrun mcpbridge` and copies the data from `content` into `structuredContent`, making Xcode's MCP tools fully compatible with all MCP clients.

## Architecture

```
┌─────────────┐    MCP Protocol    ┌──────────────────┐   MCP Protocol   ┌────────────┐    XPC    ┌─────────┐
│   Cursor    │ ◄────────────────► │  xcodemcpwrapper │ ◄──────────────► │ mcpbridge  │ ◄───────► │  Xcode  │
│ (MCP Client)│                    │  (This Project)  │                  │  (Bridge)  │           │  (IDE)  │
└─────────────┘                    └──────────────────┘                  └────────────┘           └─────────┘
```

## Project Status

**✅ COMPLETE - 2026-02-08**

| Phase | Tasks | Status |
|-------|-------|--------|
| Phase 1: Foundation & Scaffolding | 6/6 | ✅ Complete |
| Phase 2: Core Bridge Implementation | 7/7 | ✅ Complete |
| Phase 3: Response Transformation Engine | 10/10 | ✅ Complete |
| Phase 4: Edge Case Handling | 9/9 | ✅ Complete |
| Phase 5: Testing & Verification | 14/14 | ✅ Complete |
| Phase 6: Packaging & Distribution | 8/8 | ✅ Complete |
| Phase 7: Documentation | 11/11 | ✅ Complete |
| Phase 8: Documentation Publishing | 2/2 | ✅ Complete |
| Phase 10: Web UI Dashboard | 1/1 | ✅ Complete |
| **Total** | **68/68** | **✅ 100%** |

### Metrics

- **Test Coverage:** 91.62%
- **Performance:** <0.01ms overhead per transformation (0.0023ms avg)
- **Memory:** <10MB footprint
- **Lines of Code:** ~400 Python + 2000+ lines documentation

## Project Structure

```
/
├── AGENTS.md              # This file - project context for AI agents
├── README.md              # Main project README
├── CHANGELOG.md           # Version history
├── pyproject.toml         # Python package configuration
├── Makefile               # Common development tasks
├── scripts/
│   ├── install.sh         # Installation script
│   ├── pick_next_task.py  # Task selection helper
│   └── calc_progress.py   # Progress calculator
├── src/mcpbridge_wrapper/
│   ├── __init__.py        # Package init
│   ├── __main__.py        # Main entry point
│   ├── bridge.py          # Subprocess bridge management
│   ├── transform.py       # Response transformation engine
│   ├── cli.py             # CLI entry point
│   └── webui/             # Optional Web UI dashboard
│       ├── server.py      # FastAPI server
│       ├── metrics.py     # Metrics collection
│       ├── audit.py       # Audit logging
│       ├── config.py      # Web UI configuration
│       └── static/        # Dashboard frontend
├── tests/
│   ├── unit/              # Unit tests (181+ tests)
│   │   ├── test_bridge.py
│   │   ├── test_transform.py
│   │   ├── test_main.py
│   │   └── conftest.py
│   └── integration/       # Integration tests
│       ├── test_e2e.py
│       └── test_performance.py
├── config/                # Configuration templates
│   ├── cursor-mcp.json
│   ├── claude-code.txt
│   └── codex-cli.txt
├── docs/                  # Documentation
│   ├── installation.md
│   ├── webui-setup.md     # Web UI dashboard guide
│   ├── cursor-setup.md
│   ├── claude-setup.md
│   ├── codex-setup.md
│   ├── troubleshooting.md
│   ├── architecture.md
│   └── tools-reference.md
└── SPECS/                 # Specifications and task tracking
    ├── Workplan.md        # Master task tracker (65 tasks)
    ├── COMMANDS/          # Workflow commands
    ├── ARCHIVE/           # Completed task artifacts (65 archived)
    └── INPROGRESS/        # Current task tracking
```

## Technology Stack

- **Python 3.7+** - Wrapper implementation (tested on 3.10.19)
- **Xcode 26.3+** - Required for MCP bridge functionality
- **MCP Protocol** - Model Context Protocol for AI tool integration
- **pytest** - Testing framework (~92% coverage)
- **ruff** - Linting and formatting
- **mypy** - Type checking

## Quick Start

### Prerequisites

1. Xcode 26.3 or later
2. Enable Xcode Tools MCP Server:
   - Open **Xcode > Settings** (`⌘,`)
   - Select **Intelligence** in the sidebar

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SoundBlaster/XcodeMCPWrapper](https://github.com/SoundBlaster/XcodeMCPWrapper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
