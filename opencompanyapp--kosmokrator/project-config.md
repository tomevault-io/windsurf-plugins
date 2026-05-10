---
trigger: always_on
description: AI coding agent for the terminal. Mythology-themed CLI built with PHP 8.4, Symfony Console, and a dual renderer (TUI/ANSI).
---

# KosmoKrator

AI coding agent for the terminal. Mythology-themed CLI built with PHP 8.4, Symfony Console, and a dual renderer (TUI/ANSI).

## Quick Start

```bash
bin/kosmokrator              # Run with auto-detected renderer (TUI if available, ANSI fallback)
bin/kosmokrator --renderer=ansi   # Force ANSI mode
bin/kosmokrator --no-animation    # Skip the animated intro
```

## Architecture

```
bin/kosmokrator → Kernel → AgentCommand → AgentSessionBuilder → AgentLoop (REPL)
                                            ├── ToolExecutor → tools + PermissionEvaluator
                                            ├── ContextManager → compaction, pruning, system prompt
                                            ├── StuckDetector → headless loop convergence
                                            ├── LLM client (AsyncLlmClient or PrismService)
                                            ├── UIManager → TuiRenderer | AnsiRenderer
                                            ├── ToolRegistry → coding, shell, task, memory, session, Lua, and subagent tools
                                            └── SubagentOrchestrator → parallel child agents
```

### Key directories

- `src/Agent/` — Agent core: AgentLoop (REPL orchestrator), ToolExecutor, ContextManager, StuckDetector, subagent system
- `src/LLM/` — LLM clients: AsyncLlmClient (Amp HTTP, async), PrismService (Prism PHP, sync), RetryableLlmClient (decorator)
  - **prism-relay boundary**: Provider-specific logic (SSE parsing quirks, `stream_options` support, usage extraction formats, `finish_reason` mapping, error normalization, prompt caching strategies) belongs in the `opencompany/prism-relay` package, NOT here. KosmoKrator's LLM layer should only contain agent-level orchestration: retry policy, streaming-to-UI bridging, cancellation handling.
- `src/UI/` — Rendering layer with split interface hierarchy
  - `UI/Tui/` — Symfony TUI renderer: TuiRenderer, TuiModalManager (dialogs), TuiAnimationManager (breathing/spinners), SubagentDisplayManager, widgets
  - `UI/Ansi/` — Pure ANSI fallback: AnsiRenderer, MarkdownToAnsi (with Handler/ for table/list extraction), AnsiTableRenderer
  - `UI/Diff/` — Unified diff rendering with word-level highlighting
  - `UI/Theme.php` — Shared color palette, tool icons, context bar
  - `UI/AgentDisplayFormatter.php` — Shared agent display utilities (used by both renderers)
  - `UI/AgentTreeBuilder.php` — Builds agent tree from orchestrator stats
- `src/Tool/` — Tool implementations in `Coding/`, permission system in `Permission/`
- `src/Command/` — AgentCommand (main REPL/headless), SetupCommand, ConfigCommand, AuthCommand, gateway/integration commands, slash commands in `Slash/`, power commands in `Power/`
- `src/Sdk/` — Stable embeddable PHP SDK over the headless runtime: AgentBuilder, Agent, events, renderers, and configuration helpers
- `src/Acp/` — Agent Client Protocol stdio server, JSON-RPC transport, ACP renderer, and session/MCP overlay bridge
- `src/Integration/` — OpenCompany integration catalog, runtime, docs, credential resolution, and Lua invocation helpers
- `src/Mcp/` — MCP config compatibility, stdio client, trust/permission checks, headless runtime, and Lua bridge
- `src/Lua/` — Lua sandbox, docs service, and native tool bridge
- `src/Session/` — SQLite persistence: sessions, messages, memories, settings
- `src/Task/` — Task tracking system with tool integrations

### Rendering

`RendererInterface` is composed from 5 focused sub-interfaces:
- `CoreRendererInterface` — lifecycle, streaming, status, phase transitions
- `ToolRendererInterface` — tool call/result display, permission prompts
- `DialogRendererInterface` — settings, session picker, plan approval, user questions
- `ConversationRendererInterface` — history clear/replay
- `SubagentRendererInterface` — subagent status, spawn/batch display, dashboard

Four renderers implement the full interface:
- **TuiRenderer** — Interactive Symfony TUI with widgets, Revolt event loop, EditorWidget for multi-line input. Delegates to TuiModalManager (overlay dialogs), TuiAnimationManager (breathing/spinners/phase), and SubagentDisplayManager (subagent lifecycle).
- **AnsiRenderer** — Pure ANSI escape codes, readline input, MarkdownToAnsi for response formatting
- **HeadlessRenderer** — Non-interactive stdout/stderr renderer for `-p`, JSON, and stream-json runs
- **NullRenderer** — Silent renderer for subagents and tests

Both use `Theme` for colors and `KosmokratorTerminalTheme` for syntax highlighting via tempest/highlight.

### Agent internals

AgentLoop is a thin orchestrator that delegates to:
- **ToolExecutor** — permission checking, concurrent tool execution partitioning, subagent spawn/batch UI
- **ContextManager** — pre-flight context window checks, LLM-based compaction, system prompt refresh
- **StuckDetector** — rolling-window repetition detection for headless subagent loops (nudge → final notice → force return)

Session setup is handled by **AgentSessionBuilder**, which wires all dependencies (LLM client, permissions, tools, subagent infrastructure) and returns an **AgentSession** value object.

## Development

```bash
composer install
php vendor/bin/phpunit

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenCompanyApp/kosmokrator](https://github.com/OpenCompanyApp/kosmokrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
