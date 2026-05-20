---
trigger: always_on
description: Handles events from LangBot pipeline.
---

# AGENTS.md

This file is for guiding code agents (like Claude Code, GitHub Copilot, OpenAI Codex, etc.) to work in the langbot-plugin-sdk project.

**IMPORTANT**: This document may contain outdated information and may differ from the actual content. Please refer to the actual situation for accuracy.

## Project Overview

LangBot Plugin SDK is the infrastructure for LangBot's plugin system, providing:
- **Plugin SDK**: Python APIs and interfaces for plugin development
- **Plugin Runtime**: Execution environment managing plugin lifecycle
- **CLI Tools**: `lbp` command for scaffolding, building, and debugging plugins
- **Communication Protocol**: Bidirectional action-based protocol (stdio/WebSocket)

The SDK enables developers to extend LangBot with custom:
- **Commands**: User-triggered actions (e.g., `!weather tokyo`)
- **Tools**: LLM-callable functions for AI agents (e.g., web search, database queries)
- **Event Listeners**: Handlers for message events (e.g., auto-reply, content filtering)

## Technology Stack

- **Language**: Python 3.10+
- **Async Framework**: asyncio with async/await
- **Data Validation**: Pydantic v2 with type hints
- **Communication**: WebSockets, stdio
- **CLI**: argparse with Jinja2 templates
- **Package Manager**: uv (recommended) or pip

## Project Structure

```
langbot-plugin-sdk/
├── src/langbot_plugin/
│   ├── api/                     # SDK APIs for plugin developers
│   │   ├── definition/          # Base classes
│   │   │   ├── plugin.py        # BasePlugin
│   │   │   └── components/      # Component base classes
│   │   │       ├── base.py      # BaseComponent
│   │   │       ├── command/     # Command component
│   │   │       ├── tool/        # Tool component
│   │   │       └── common/      # EventListener
│   │   ├── entities/            # Data models
│   │   │   ├── context.py       # ExecuteContext, EventContext
│   │   │   ├── events.py        # Event types
│   │   │   └── builtin/         # Platform entities
│   │   │       ├── platform/    # MessageChain, components
│   │   │       ├── command/     # CommandReturn
│   │   │       └── provider/    # Session, Conversation
│   │   └── proxies/             # API proxy classes
│   │       ├── langbot_api.py   # LangBotAPIProxy
│   │       └── query_based.py   # QueryBasedAPIProxy
│   ├── runtime/                 # Plugin runtime system
│   │   ├── plugin/              # Plugin management
│   │   │   ├── mgr.py           # PluginManager
│   │   │   ├── container.py     # PluginContainer
│   │   │   └── installer.py    # Plugin installation
│   │   ├── io/                  # Communication layer
│   │   │   ├── handler.py       # Action handler
│   │   │   ├── stdio.py         # Stdio transport
│   │   │   └── websocket.py    # WebSocket transport
│   │   └── event/               # Event dispatching
│   ├── cli/                     # Command-line tools
│   │   ├── main.py              # lbp entrypoint
│   │   ├── init.py              # Plugin initialization
│   │   ├── gen/                 # Component generation
│   │   └── run.py               # Plugin debugging
│   ├── entities/                # Internal data structures
│   │   ├── io/                  # Communication protocol
│   │   └── plugin/              # Plugin metadata
│   └── utils/                   # Utilities
│       ├── discover/            # Component discovery
│       └── network/             # Network helpers
├── docs/                        # Documentation
├── pyproject.toml               # Python project config
└── README.md
```

## Plugin System Architecture

### Communication Model

```
┌─────────────────┐         ┌──────────────────┐
│    LangBot      │         │  Plugin Process  │
│                 │◄───────►│                  │
│  Plugin Runtime │ stdio/  │  BasePlugin      │
│                 │ WebSocket│  Components      │
└─────────────────┘         └──────────────────┘
```

**Transport Modes**:
1. **Stdio Mode** (default):
   - Plugin runs as subprocess of LangBot
   - Communication via stdin/stdout
   - Used in personal/lightweight deployments

2. **WebSocket Mode**:
   - Plugin connects to Runtime via WebSocket
   - Used in containerized/production environments
   - Enables remote plugin debugging

### Plugin Lifecycle

1. **Discovery**: Runtime scans plugin directories, reads manifests
2. **Installation**: Download from GitHub/marketplace, install dependencies
3. **Loading**: Import plugin module, instantiate `BasePlugin` class
4. **Initialization**: Call `async initialize()`, send config
5. **Registration**: Register components (Commands, Tools, EventListeners)
6. **Ready**: Plugin accepts events and requests
7. **Termination**: Cleanup and shutdown

### Component Architecture

```
BasePlugin
  ├── Command (1+ per plugin)
  │   └── Subcommand handlers
  ├── Tool (0+ per plugin)
  │   └── call() method
  └── EventListener (1 per plugin)
      └── Event handlers
```

## SDK Development

### Setup Environment

```bash
# Install dependencies
uv sync --dev
# Or with pip
pip install -e .

# Run CLI
uv run lbp --help
# Or
python -m langbot_plugin.cli.main --help
```

### Plugin Development Workflow

#### 1. Initialize Plugin

```bash
lbp init MyPlugin
cd MyPlugin
```

Creates:
- `manifest.yaml`: Plugin metadata and configuration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [langbot-app/langbot-plugin-sdk](https://github.com/langbot-app/langbot-plugin-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
