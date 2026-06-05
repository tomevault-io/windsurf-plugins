---
trigger: always_on
description: > **Note:** This is a quick-reference guide for AI assistants. For comprehensive documentation, see:
---

# Git-Iris Developer Guide

> **Note:** This is a quick-reference guide for AI assistants. For comprehensive documentation, see:
>
> - **[Architecture Documentation](docs/architecture/)** — System design, patterns, and data flow
> - **[Theme System Documentation](docs/themes/)** — SilkCircuit design language and theming
> - **[Studio Internals](docs/studio-internals/)** — Deep dive into TUI implementation
> - **[Extension Guide](docs/extending/)** — Creating new capabilities, tools, and modes

## Architecture Overview

Git-Iris uses an agent-first architecture powered by **Iris**, an LLM-driven agent built on the [Rig framework](https://docs.rs/rig-core). Iris dynamically explores codebases using tool calls rather than dumping all context upfront.

### Core Principles

- **LLM-First**: The LLM makes all intelligent decisions—we avoid deterministic heuristics
- **Tool-Based Context**: Iris gathers precisely what she needs via tool calls
- **Unified Interface**: Studio provides a single TUI for all capabilities
- **Event-Driven State**: Reducer-centric event flow for predictable, testable state management

## Project Structure

```
src/
├── agents/                       # Agent framework (core of Git-Iris)
│   ├── iris.rs                   # Main agent implementation
│   ├── core.rs                   # Backend abstraction (OpenAI/Anthropic/Google)
│   ├── context.rs                # TaskContext and shared agent context
│   ├── provider.rs               # Provider-specific agent builders (caching, models)
│   ├── setup.rs                  # IrisAgentService entry point
│   ├── status.rs                 # Real-time status tracking
│   ├── status_messages.rs        # Witty status messages via fast model
│   ├── debug.rs                  # Debug mode output formatting
│   ├── debug_tool.rs             # Tool wrapper for debug instrumentation
│   ├── output_validator.rs       # JSON recovery for malformed responses
│   ├── capabilities/             # Task-specific prompts (TOML, 8 total)
│   │   ├── commit.toml           # Commit message generation
│   │   ├── review.toml           # Structured code review (findings, severity)
│   │   ├── pr.toml               # PR description generation
│   │   ├── changelog.toml        # Changelog generation
│   │   ├── release_notes.toml    # Release notes
│   │   ├── chat.toml             # Interactive chat capability
│   │   ├── semantic_blame.toml   # "Why does this code exist?"
│   │   └── verify.toml           # Critic verification pass (internal)
│   └── tools/                    # Tools Iris can use
│       ├── registry.rs           # CORE_TOOLS list + attach_core_tools! macro
│       ├── common.rs             # Shared tool utilities (repo root, schemas)
│       ├── git.rs                # git_diff, git_log, git_status, git_show,
│       │                         # git_changed_files, git_blame, git_repo_info
│       ├── file_read.rs          # File content reading and targeted excerpts
│       ├── code_search.rs        # Pattern searching
│       ├── repo_map.rs           # Ranked codebase orientation map
│       ├── static_analysis.rs    # Direct linter runs (rust/python/js/go)
│       ├── docs.rs               # Project documentation (README, CLAUDE.md)
│       ├── workspace.rs          # Iris's notes and task tracking
│       ├── parallel_analyze.rs   # Concurrent subagent processing
│       └── content_update.rs     # Chat tools that update commit/PR/review
│
├── studio/                       # Iris Studio TUI (Ratatui-based)
│   ├── app/                      # Main event loop and app coordination
│   │   ├── mod.rs                # StudioApp lifecycle
│   │   └── agent_tasks.rs        # Agent task spawning and orchestration
│   ├── state/                    # Centralized state for all modes
│   │   ├── mod.rs                # StudioState root and helpers
│   │   ├── chat.rs               # ChatState (messages, streaming, tools)
│   │   └── modes.rs              # ExploreState, CommitState, ReviewState, etc.
│   ├── reducer/                  # Reducer-centric state transitions
│   │   ├── mod.rs                # Top-level reduce() dispatch
│   │   ├── agent.rs              # Agent task events
│   │   ├── content.rs            # Content updates (commit, PR, review)
│   │   ├── git.rs                # Git operation events
│   │   ├── modal.rs              # Modal open/close transitions
│   │   ├── navigation.rs         # Mode switching and panel focus
│   │   ├── settings.rs           # Settings modal state
│   │   └── ui.rs                 # UI-level events (resize, scroll)
│   ├── events.rs                 # StudioEvent, AgentTask, SideEffect enums
│   ├── history.rs                # Audit trail and session persistence
│   ├── layout.rs                 # Layout calculations and panel sizing
│   ├── theme.rs                  # Studio-specific style derivation
│   ├── utils.rs                  # Shared rendering helpers
│   ├── components/               # Reusable UI components
│   │   ├── code_view.rs          # Syntax-highlighted source display
│   │   ├── diff_view.rs          # Unified diff rendering with hunks
│   │   ├── file_tree.rs          # Directory navigation with git status

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hyperb1iss/git-iris](https://github.com/hyperb1iss/git-iris) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
