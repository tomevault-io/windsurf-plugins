---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Architecture Overview

Quicksilver is an Elixir-native agentic framework with tool-calling capabilities. The key architectural pattern is:

```
Terminal Interface (though any interface can be used)
    ↓
ToolAgent (GenServer, multiple agents can exist and be used)
    ↓
Backend (LlamaCpp) → External LLM Server
    ↓
Tools (Registry + Individual Tools)
```

### Critical Design Patterns

**1. Agentic Loop with Per-Iteration Timeouts**

The ToolAgent (`lib/quicksilver/agents/tool_agent.ex`) implements an iterative reasoning loop:
- Max 50 iterations per task (configurable)
- Each iteration has a 5-minute timeout (per-iteration, not total)
- Overall 10-minute safety timeout on the GenServer call
- Uses `Task.async` + `Task.yield` for per-iteration timeout control
- On each iteration: prompt LLM → parse response → execute tool → add to history → repeat

**2. Tool System Architecture**

Tools are defined via the `Quicksilver.Tools.Behaviour`:
- `name/0` - Unique identifier
- `description/0` - LLM-facing description
- `parameters_schema/0` - JSON schema for arguments
- `execute/2` - Implementation (args, context)

The Registry (`lib/quicksilver/tools/registry.ex`) is an Agent that stores tool modules and handles execution.

The Formatter (`lib/quicksilver/tools/formatter.ex`) converts tools to LLM prompts and parses tool calls from various JSON formats.

**3. Approval System for Destructive Operations**

Located in `lib/quicksilver/approval/`:
- `Policy` - Defines which tools require approval
- `Interactive` - Shows diffs and prompts user for [A]pprove/[R]eject/[Q]uit

Context includes `:approval_policy` which is checked before executing write tools.

**4. Backend Abstraction**

`lib/backends/backend.ex` defines the behaviour. Currently only `LlamaCpp` is implemented, which:
- Manages llama.cpp server lifecycle (starts if not running)
- Uses HTTP API for completions
- Configured in `config/config.exs`

### Supervision Tree

```elixir
Quicksilver.Supervisor
├── Registry (for agent processes)
├── Quicksilver.Backends.LlamaCpp (GenServer)
├── Quicksilver.Tools.Registry (Agent)
├── Quicksilver.Agents.ToolAgent (GenServer)
└── Quicksilver.Agents.Manager (GenServer)
```

All supervised processes start on application boot. Tools are registered after supervisor starts successfully.

## Available Tools (As of Current Implementation)

**Read-only:**
- `read_file` - Read file contents
- `search_files` - Search codebase (ripgrep/grep)
- `list_files` - List files with glob patterns

**Write (require approval):**
- `create_file` - Create new files
- `edit_file` - Edit existing files (uses exact string replacement with uniqueness validation)

**Utility:**
- `run_tests` - Execute `mix test` and return results

All write tools create `.backup.timestamp` files before modifications.

## Adding New Tools

1. Create module in `lib/quicksilver/tools/`
2. Implement `@behaviour Quicksilver.Tools.Behaviour`
3. Add to `lib/quicksilver/tools.ex` alias and `register_default_tools/0`
4. If destructive: integrate with approval system via context

## Agent Execution Context

When tools execute, they receive a `context` map with:
- `:workspace_root` - Base directory for file operations
- `:approval_policy` - Policy struct for approval checks

The ToolAgent automatically injects the default approval policy into context on each iteration.

## Terminal Interface

`lib/interfaces/terminal.ex` provides the interactive chat:
- Commands: `help`, `tools`, `agents`, `agent <name>`, `history`, `clear`, `exit`
- Multi-agent switching with conversation history preservation
- Routes to ToolAgent's `execute_task/3`

## Configuration

`config/config.exs` contains llama.cpp server settings:
- Server binary path
- Model path and filename
- Port, threads, context size, GPU layers

Update these paths to match your local setup.

## Key Behaviors to Maintain

**Tool Uniqueness Validation**: The `edit_file` tool requires `old_string` to appear exactly once in the file. This prevents ambiguous edits.

**Backup Creation**: Always create backups before destructive operations (pattern: `path.backup.timestamp`).

**Per-Iteration Timeouts**: When modifying ToolAgent, maintain the pattern of per-iteration timeouts rather than total task timeouts to allow long-running multi-tool tasks.

**Tool Result Formatting**: Use `Formatter.format_tool_result/2` to ensure consistent formatting of tool outputs in conversation history.

## Testing Notes

- Unit tests in `test/` directory
- Integration tests in `test_editing.exs` (tests file creation/editing with auto-approve policy)
- Run terminal with `mix run start_chat.exs` for manual testing

## Common Pitfalls

1. **Don't use total timeouts** - The GenServer call uses `:infinity` (with 10min safety timeout), individual iterations timeout at 5min
2. **Tool registration timing** - Tools must be registered after supervision tree starts
3. **Context propagation** - When adding recursive calls in ToolAgent, ensure `per_iteration_timeout` is passed through

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JediLuke/quicksilver](https://github.com/JediLuke/quicksilver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
