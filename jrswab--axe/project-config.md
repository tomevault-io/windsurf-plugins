---
trigger: always_on
description: A lightweight CLI for running single-purpose LLM agents. Think `make` for AI — define agents in TOML, give them focused context via SKILL.md files, and trigger them from anywhere (cron, git hooks, pipes, file watchers).
---

# Axe — AGENTS.md

## What This Is

A lightweight CLI for running single-purpose LLM agents. Think `make` for AI — define agents in TOML, give them focused context via SKILL.md files, and trigger them from anywhere (cron, git hooks, pipes, file watchers).

## Design Philosophy

- **Small context windows by design.** Each agent gets only what it needs. Sub-agents return results, not conversations. This is the core value prop — fight context bloat at every level.
- **Unix citizen.** Stdout is clean and pipeable. Debug goes to stderr. Exit codes are meaningful. Stdin is always accepted. Axe composes with existing tools, it doesn't replace them.
- **Executor, not scheduler.** Axe runs agents. Triggering is the user's job (cron, entr, git hooks, webhooks). No built-in daemon, no watch mode, no event loop.
- **Single binary, zero runtime.** Go. No interpreters, no node_modules, no Docker required. `go build` and ship.
- **Config over code.** Agent definitions are TOML, not Go. Users should never touch source to define or modify agents.

## Non-Obvious Constraints

- **SKILL.md is a community format** — not invented here. Treat it as an external standard. Don't add axe-specific extensions to the format itself; put axe-specific config in TOML.
- **models.dev format for model strings** — always `provider/model-name`. Don't invent a custom format.
- **XDG Base Directory spec** — config, data, and cache go where XDG says. No dotfiles in $HOME.
- **Sub-agents are opaque to parents.** A parent never sees a sub-agent's internal turns, tool calls, or files. Only the final text result crosses the boundary. This is intentional — don't leak internals upward.
- **Depth limits are safety rails, not features.** Default 3, hard max 5. If someone needs more depth, the design is wrong, not the limit.

## What Good Contributions Look Like

- Tests for every new package (table-driven, no mocks when avoidable — use the real thing or `testutil` helpers)
- Errors that help the user fix the problem, not just describe it
- No global state — pass dependencies explicitly
- Flags override TOML overrides defaults. Resolution order matters everywhere.
- When in doubt, print nothing. Axe output MUST be safe to pipe.

## Directory Structure

```text
axe/
├── cmd/                    # CLI commands (run, agents, config, gc, version)
├── internal/
│   ├── agent/             # Agent TOML loading and validation
│   ├── config/            # Global config.toml handling
│   ├── envinterp/         # ${VAR} expansion for MCP headers
│   ├── mcpclient/         # Model Context Protocol client
│   ├── memory/            # Persistent memory system
│   ├── provider/          # LLM provider implementations (Anthropic, OpenAI, Ollama, OpenCode, Bedrock)
│   ├── refusal/           # LLM refusal detection
│   ├── resolve/           # Context resolution (workdir, skill, files, stdin)
│   ├── testutil/          # Test helpers and mock servers
│   ├── tool/              # Built-in tools (file ops, shell, web)
│   ├── toolname/          # Tool name constants
│   └── xdg/               # XDG Base Directory support
├── docs/
│   ├── design/            # Architecture docs
│   └── plans/             # Implementation specs
├── examples/              # Example agents (code-reviewer, commit-msg, summarizer)
└── main.go
```

## Key Entry Points

### CLI Commands (`cmd/`)

- **run.go** - Main execution: loads agent, resolves context, calls provider, handles conversation loop (max 50 turns), executes tools (parallel by default), manages memory
- **agents.go** - Agent management: list, show, init (scaffold), edit (opens $EDITOR)
- **config.go** - Config init (creates XDG dirs, embeds sample skill), path display
- **gc.go** - Memory garbage collection with LLM-assisted pattern detection or fallback to max_entries
- **root.go** - Root command, exit code mapping (0=success, 1=runtime error, 2=config error)

### Core Packages

- **internal/agent** - `Load()` parses TOML, `Validate()` checks required fields and constraints, `Scaffold()` generates template
- **internal/provider** - `Provider` interface with `Send(ctx, Request) (Response, error)`. Implementations: Anthropic (messages API), OpenAI (chat completions), Ollama (local), OpenCode (multi-route: Claude/messages, GPT/responses, others/chat/completions), Bedrock (AWS Converse API)
- **internal/tool** - Registry pattern. Built-in tools: list_directory, read_file (with pagination), write_file, edit_file (find/replace), run_command (via sh -c), url_fetch (HTML stripping), web_search, call_agent (sub-agent delegation). All file tools sandboxed to workdir.
- **internal/resolve** - Resolution order: flags > TOML > env vars > defaults. Handles tilde/env var expansion, glob patterns (simple and **), binary file detection, symlink escape prevention.
- **internal/memory** - Timestamped markdown entries. `AppendEntry()` adds task/result, `LoadEntries()` loads last N, `TrimEntries()` keeps last N, `CountEntries()` returns total.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jrswab/axe](https://github.com/jrswab/axe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
