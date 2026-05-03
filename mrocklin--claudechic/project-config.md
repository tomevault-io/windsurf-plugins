---
trigger: always_on
description: A stylish terminal UI for Claude Code, built with Textual and wrapping the `claude-agent-sdk`.
---

# Claude Chic

A stylish terminal UI for Claude Code, built with Textual and wrapping the `claude-agent-sdk`.

## Run

```bash
uv run claudechic
uv run claudechic --resume     # Resume most recent session
uv run claudechic -s <uuid>    # Resume specific session
```

Requires Claude Code to be logged in with a Max/Pro subscription (`claude /login`).

## File Map

```
claudechic/
├── __init__.py        # Package entry, exports ChatApp
├── __main__.py        # CLI entry point
├── agent.py           # Agent class - SDK connection, history, permissions, state
├── analytics.py       # PostHog analytics - fire-and-forget event tracking
├── agent_manager.py   # AgentManager - coordinates multiple concurrent agents
├── app.py             # ChatApp - main application, event handlers
├── commands.py        # Slash command routing (/agent, /shell, /clear, etc.)
├── compact.py         # Session compaction - shrink old tool uses to save context
├── errors.py          # Logging infrastructure, error handling
├── file_index.py      # Fuzzy file search using git ls-files
├── formatting.py      # Tool formatting, diff rendering (pure functions)
├── history.py         # Global history loading from ~/.claude/history.jsonl
├── mcp.py             # In-process MCP server for agent control tools
├── messages.py        # Custom Textual Message types for SDK events
├── remote.py          # HTTP server for remote control (live testing)
├── permissions.py     # PermissionRequest dataclass for tool approval
├── profiling.py       # Lightweight profiling utilities (@profile decorator)
├── sampling.py        # CPU-conditional sampling profiler for high-CPU investigation
├── protocols.py       # Observer protocols (AgentObserver, AgentManagerObserver)
├── sessions.py        # Session file loading and listing (pure functions)
├── styles.tcss        # Textual CSS - visual styling
├── theme.py           # Textual theme definition
├── usage.py           # OAuth usage API fetching (rate limits)
├── features/
│   ├── __init__.py    # Feature module exports
│   └── worktree/
│       ├── __init__.py   # Public API (list_worktrees, handle_worktree_command)
│       ├── commands.py   # /worktree command handlers
│       └── git.py        # Git worktree operations
├── processes.py       # BackgroundProcess dataclass, child process detection
├── screens/           # Full-page screens (navigation)
│   ├── chat.py        # ChatScreen - main chat UI (default screen)
│   ├── diff.py        # DiffScreen - review uncommitted changes
│   └── session.py     # SessionScreen - session browser for /resume
└── widgets/
    ├── __init__.py    # Re-exports all widgets for backward compat
    ├── prompts.py     # All prompt widgets (Selection, Question, Model, Worktree)
    ├── base/          # Protocols and base classes
    │   ├── clickable.py # ClickableLabel base class
    │   ├── tool_base.py # ToolWidgetBase class
    │   └── tool_protocol.py # ToolWidget protocol
    ├── primitives/    # Low-level building blocks
    │   ├── button.py  # Button with click handling
    │   ├── collapsible.py # QuietCollapsible
    │   ├── scroll.py  # AutoHideScroll
    │   └── spinner.py # Animated spinner
    ├── content/       # Content display widgets
    │   ├── message.py # ChatMessage, ChatInput, ThinkingIndicator
    │   ├── tools.py   # ToolUseWidget, TaskWidget, AgentToolWidget
    │   ├── diff.py    # Syntax-highlighted diff widget
    │   └── todo.py    # TodoPanel, TodoWidget
    ├── input/         # User input widgets
    │   ├── autocomplete.py # TextAreaAutoComplete
    │   └── history_search.py # HistorySearch (Ctrl+R)
    ├── layout/        # Structural/container widgets
    │   ├── chat_view.py # ChatView - renders agent messages
    │   ├── sidebar.py # AgentSidebar, AgentItem, WorktreeItem
    │   ├── footer.py  # StatusFooter, AutoEditLabel, ModelLabel
    │   ├── indicators.py # IndicatorWidget, CPUBar, ContextBar, ProcessIndicator
    │   └── processes.py # ProcessPanel, ProcessItem
    ├── reports/       # In-page report widgets
    │   ├── context.py # ContextReport - visual 2D grid
    │   └── usage.py   # UsageReport, UsageBar
    └── modals/        # Modal screen overlays
        ├── profile.py # ProfileModal - profiling stats
        └── process_modal.py # ProcessModal

tests/
├── __init__.py        # Package marker
├── conftest.py        # Shared fixtures (wait_for)
├── test_app.py        # E2E tests with real SDK
├── test_app_ui.py     # App UI tests without SDK
├── test_autocomplete.py # Autocomplete widget tests
├── test_file_index.py # Fuzzy file search tests
└── test_widgets.py    # Pure widget tests
```

## Architecture

### Module Boundaries

**Pure functions (no UI dependencies):**
- `formatting.py` - Tool header formatting, diff rendering, language detection
- `sessions.py` - Session file I/O, listing, filtering
- `file_index.py` - Fuzzy file search, git ls-files integration
- `compact.py` - Session compaction to reduce context window usage
- `usage.py` - OAuth API for rate limit info

**Agent layer (no UI dependencies):**
- `agent.py` - `Agent` class owns SDK client, message history, permissions, state
- `agent_manager.py` - Coordinates multiple agents, switching, lifecycle

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mrocklin/claudechic](https://github.com/mrocklin/claudechic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
