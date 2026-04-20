---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Core Development Tasks

- **Install dependencies**: `make install` (requires uv and pre-commit)
- **Run all checks**: `make all` or `pre-commit run --all-files`
- **Run tests**: `make test`
- **Build docs**: `make docs` or `make docs-serve` (local development)

### Single Test Commands

- **Run specific test**: `uv run pytest tests/test_agent.py::test_function_name -v`
- **Run test file**: `uv run pytest tests/test_agent.py -v`
- **Run with debug**: `uv run pytest tests/test_agent.py -v -s`

## Project Architecture

### Repository Layout

- `pydantic_deep/` — Core library (agent, deps, toolsets, capabilities, processors)
- `apps/cli/` — CLI + TUI application (Textual-based terminal AI assistant)
- `apps/deepresearch/` — Full-featured research reference app
- `tests/` — Unit tests
- `docs/` — Documentation source (MkDocs)

### Core Components

**Agent Factory (`pydantic_deep/agent.py`)**
- `create_deep_agent()`: Main factory function for creating configured agents
- `create_default_deps()`: Helper for creating DeepAgentDeps with sensible defaults
- Built on top of pydantic-ai's Agent class
- Requires pydantic-ai>=1.77.0

**Dependencies (`pydantic_deep/deps.py`)**
- `DeepAgentDeps`: Dataclass holding agent dependencies (backend, working_dir, skills_dirs, subagents)
- Passed to agent.run() for runtime configuration

**Backends (from [pydantic-ai-backend](https://github.com/vstorm-co/pydantic-ai-backend))**
- `BackendProtocol`: Interface for file storage backends
- `StateBackend`: In-memory file storage (for testing, ephemeral use)
- `LocalBackend`: Real filesystem operations
- `DockerSandbox`: Isolated Docker container execution
- `CompositeBackend`: Combines multiple backends with routing

**Toolsets (`pydantic_deep/toolsets/`)**
- `TodoToolset`: Task planning and tracking tools (read_todos, write_todos) - from [pydantic-ai-todo](https://github.com/vstorm-co/pydantic-ai-todo)
- `create_console_toolset`: File operations (ls, read, write, edit, glob, grep, execute) - from [pydantic-ai-backend](https://github.com/vstorm-co/pydantic-ai-backend)
- `SubAgentToolset`: Spawn and delegate to subagents - from [subagents-pydantic-ai](https://github.com/vstorm-co/subagents-pydantic-ai)
- `SkillsToolset`: Load and use skill definitions from markdown files
- Web search and fetch use pydantic-ai built-in `WebSearch()` and `WebFetch()` tools

**Subagents (from [subagents-pydantic-ai](https://github.com/vstorm-co/subagents-pydantic-ai))**
- `create_subagent_toolset()`: Factory function to create subagent toolsets
- `get_subagent_system_prompt()`: Generate system prompt for subagent tools
- Dual-mode execution: sync (blocking) or async (background)
- Task management: check_task, list_active_tasks, soft_cancel_task, hard_cancel_task
- Types: `SubAgentConfig`, `CompiledSubAgent`, `TaskHandle`, `TaskStatus`, `TaskPriority`

**Processors (from [summarization-pydantic-ai](https://github.com/vstorm-co/summarization-pydantic-ai))**
- `SummarizationProcessor`: LLM-based conversation summarization for token management
- `SlidingWindowProcessor`: Zero-cost message trimming without LLM calls
- `create_summarization_processor()`: Factory function for summarization processors
- `create_sliding_window_processor()`: Factory function for sliding window processors

**Types (`pydantic_deep/types.py`)**
- Pydantic models for all data structures
- `FileData`, `FileInfo`, `WriteResult`, `EditResult`, `GrepMatch`
- `Todo`, `SubAgentConfig`, `CompiledSubAgent`
- `Skill`, `SkillDirectory`, `SkillFrontmatter`
- `ResponseFormat`: Alias for structured output specification

**Checkpointing (`pydantic_deep/toolsets/checkpointing.py`)**
- `Checkpoint`: Immutable snapshot of conversation state (id, label, turn, messages, metadata)
- `CheckpointStore`: Protocol for storage backends (save, get, list_all, remove, etc.)
- `InMemoryCheckpointStore`: Default in-memory store
- `FileCheckpointStore`: Persistent JSON file store
- `CheckpointMiddleware`: Auto-checkpoint capability extending `AbstractCapability` (every_tool, every_turn, manual_only)
- `CheckpointToolset`: Agent tools (save_checkpoint, list_checkpoints, rewind_to)
- `RewindRequested`: Exception for app-level rewind (propagates out of agent.run())
- `fork_from_checkpoint()`: Utility for session forking

**Agent Teams (`pydantic_deep/toolsets/teams.py`)**
- `SharedTodoItem`: Task with assignment, dependencies, and status tracking
- `SharedTodoList`: Asyncio-safe shared TODO list with claiming and dependency blocking
- `TeamMessage`: Message between team members
- `TeamMessageBus`: Peer-to-peer message bus using asyncio.Queue per agent
- `TeamMember`: Member definition (name, role, description, instructions, model)
- `TeamMemberHandle`: Runtime handle to a running team member
- `AgentTeam`: Coordinator — spawn, assign, broadcast, wait_all, dissolve
- `create_team_toolset()`: Factory for team management tools (spawn_team, assign_task, check_teammates, message_teammate, dissolve_team)

**Output Styles (`pydantic_deep/styles.py`)**
- `OutputStyle`: Dataclass (name, description, content)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vstorm-co/pydantic-deepagents](https://github.com/vstorm-co/pydantic-deepagents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
