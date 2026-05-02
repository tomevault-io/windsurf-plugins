---
trigger: always_on
description: **pai-agent-sdk** is an application framework for building AI agents with [Pydantic AI](https://ai.pydantic.dev/). It provides environment abstractions, session management, and hierarchical agent patterns.
---

## Project Overview

**pai-agent-sdk** is an application framework for building AI agents with [Pydantic AI](https://ai.pydantic.dev/). It provides environment abstractions, session management, and hierarchical agent patterns.

- **Language**: Python 3.11+
- **Package Manager**: uv
- **Build System**: hatchling

## Key Features

- **Environment-based Architecture**: Inject file operations, shell access, and resources via `Environment` for clean separation of concerns (LocalEnvironment, DockerEnvironment)
- **Resumable Sessions**: Export/restore `AgentContext` state for multi-turn conversations across restarts
- **Hierarchical Agents**: Subagent system with task delegation, tool inheritance, and markdown-based configuration
- **Skills System**: Markdown-based instruction files with hot reload and progressive loading
- **Human-in-the-Loop**: Built-in approval workflows for sensitive tool operations
- **Toolset Architecture**: Extensible tool system with pre/post hooks for logging, validation, and error handling
- **Resumable Resources**: Export and restore resource states (like browser sessions) across process restarts
- **Browser Automation**: Docker-based headless Chrome sandbox for safe browser automation
- **Streaming Support**: Real-time streaming of agent responses and tool executions

## Project Structure

```
pai_agent_sdk/
├── agents/                # Agent implementations
│   ├── main.py            # create_agent, stream_agent entry points
│   ├── compact.py         # Compact agent variant
│   ├── image_understanding.py  # Image understanding agent
│   ├── video_understanding.py  # Video understanding agent
│   └── models/            # Model configuration and inference
│
├── context.py             # AgentContext, ModelConfig, ToolConfig, ResumableState
│
├── environment/           # Environment management
│   ├── base.py            # Environment ABC, FileOperator, Shell, ResourceRegistry, BaseResource
│   ├── local.py           # LocalEnvironment for local filesystem
│   └── docker.py          # DockerEnvironment for container-based execution
│
├── toolsets/              # Tool implementations
│   ├── core/              # Core toolsets collection
│   │   ├── base.py        # BaseTool, Toolset, GlobalHooks (base classes)
│   │   ├── content/       # Content loading tools
│   │   ├── context/       # Context management tools (handoff)
│   │   ├── document/      # Document processing tools
│   │   ├── enhance/       # Enhancement tools (todo, thinking)
│   │   ├── filesystem/    # File system operation tools
│   │   ├── multimodal/    # Multimodal tools (read_image, read_video)
│   │   ├── shell/         # Shell command execution tools
│   │   ├── subagent/      # Subagent delegation tools
│   │   └── web/           # Web interaction tools
│   └── browser_use/       # Browser automation toolset (independent)
│
├── subagents/             # Subagent system
│   ├── config.py          # SubagentConfig parsing
│   ├── factory.py         # Subagent tool factory functions
│   └── presets/           # Built-in subagent presets
│       ├── debugger.md    # Debugging specialist
│       ├── explorer.md    # Codebase exploration specialist
│       ├── searcher.md    # Search specialist
│       └── code-reviewer.md # Code review specialist
│
├── filters/               # Message history processors
│   ├── handoff.py         # Handoff message processing
│   ├── image.py           # Image filtering
│   ├── system_prompt.py   # System prompt filtering
│   └── tool_args.py       # Tool argument fixing
│
├── sandbox/               # Sandbox environments
│   └── browser/           # Browser sandbox
│
├── skills/                # Skill definitions
│   └── checkpointing/     # Checkpointing skill
│
├── stream/                # Stream processing
├── presets.py             # Preset configurations (model settings, etc.)
├── usage.py               # Usage tracking models (InternalUsage, ExtraUsageRecord)
├── utils.py               # Utility functions
└── _logger.py             # Centralized logging

tests/                     # Test suite (pytest)
├── environment/           # Environment tests
├── filters/               # Filter tests
├── sandbox/               # Sandbox tests
├── subagents/             # Subagent tests
└── toolsets/              # Toolset tests
```

## Development Workflow

After modifying any code:

1. `make lint` - Quick formatting and auto-fix (ruff + pre-commit)
2. `make check` - Full validation (lock file, pre-commit, Pyright type checking, deptry)
3. `make test` - Run test suite with coverage

## Key Commands

| Command         | Description                                         |
| --------------- | --------------------------------------------------- |
| `make install`  | Create venv with uv and install pre-commit hooks    |
| `make lint`     | Run pre-commit linters (ruff format/lint)           |
| `make check`    | Full validation: lint + pyright + deptry            |
| `make test`     | Run pytest with coverage (inline snapshot disabled) |
| `make test-fix` | Run pytest with inline snapshot update enabled      |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [youware-labs/pai-agent-sdk](https://github.com/youware-labs/pai-agent-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
