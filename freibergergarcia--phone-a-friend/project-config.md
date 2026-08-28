---
trigger: always_on
description: Guidance for AI coding agents working in `phone-a-friend`.
---

# AGENTS.md

Guidance for AI coding agents working in `phone-a-friend`.

## What This Is

`phone-a-friend` is a TypeScript CLI for relaying prompts + repository context to coding backends (Claude, Antigravity, Codex, Gemini, Ollama, OpenCode). Available via `npm install -g @freibergergarcia/phone-a-friend` or from source. All backend `run()` methods are async (`Promise<string>`). Backends may also implement `runStream()` returning `AsyncIterable<string>` for token-level streaming.

## Project Structure

```
src/
  index.ts           Entry point — imports backends, runs CLI
  cli.ts             Commander.js CLI with subcommands
  relay.ts           Backend-agnostic relay core (relay + relayStream + reviewRelay)
  stream-parsers.ts  Stream parsers — SSE (OpenAI-compatible), NDJSON (Ollama), Claude JSON snapshots
  context.ts         RelayContext interface
  version.ts         Shared version reader
  detection.ts       Backend detection (CLI, Local, Host)
  config.ts          TOML configuration system
  doctor.ts          Health check command
  setup.ts           Interactive setup wizard
  installer.ts       Claude/OpenCode host integration installer (symlink/copy)
  theme.ts           Shared semantic theme (chalk) for CLI styling + banner
  display.ts         Display helpers (mark, formatBackendLine)
  jobs.ts            Background job manager (JSON persistence at ~/.config/phone-a-friend/jobs.json)
  sessions.ts        Relay session store (JSON persistence at ~/.config/phone-a-friend/sessions.json)
  backends/
    index.ts         Backend interface, registry, types, BackendCapabilities, spawnCli() async subprocess utility
    antigravity.ts  Google Antigravity CLI subprocess backend (`agy`, read-only, one-shot)
    claude.ts        Claude CLI subprocess backend (`claude -p`)
    codex.ts         Codex subprocess backend
    gemini.ts        Gemini subprocess backend
    ollama.ts        Ollama HTTP API backend (native fetch)
    opencode.ts      OpenCode CLI subprocess backend (`opencode run`, agentic with tool calling)
  agentic/
    index.ts         Public API — Orchestrator, TranscriptBus exports
    types.ts         AgentConfig, AgenticSessionConfig, AgentState, Message, AGENTIC_DEFAULTS
    orchestrator.ts  Main loop — spawn agents, route messages, guardrails, emit events
    session.ts       SessionManager — Claude CLI subprocess with UUID-based sessions
    bus.ts           SQLite transcript bus (better-sqlite3) — append-only session log
    queue.ts         In-memory MessageQueue for runtime routing
    events.ts        AgenticEvent discriminated union + EventChannel (push/pull bridge)
    parser.ts        @mention extraction + system prompt builder
    names.ts         Creative agent name assignment (e.g., ada.reviewer)
  tui/
    App.tsx          Root TUI component — tab bar + panel routing
    render.tsx       Ink render entry point
    StatusPanel.tsx  System info + backend detection display
    BackendsPanel.tsx Per-backend list with detail pane
    ConfigPanel.tsx  Config view + inline editing
    ActionsPanel.tsx Async-wrapped executable actions
    AgenticPanel.tsx Session browser with list view
    hooks/
      useDetection.ts    Async detection with throttled refresh
      usePluginStatus.ts Host integration install status (sync FS check)
      useAgenticSessions.ts  SQLite session loader for Agentic panel
    components/
      TabBar.tsx             Tab navigation bar
      PluginStatusBar.tsx    Persistent host integration install indicator
      Badge.tsx              Status badges (✓ ✗ ! ·)
      KeyHint.tsx            Footer keyboard hints
      ListSelect.tsx         Scrollable selectable list
tests/               Vitest tests (mirrors src/ structure, includes spawn-cli, jobs, background-relay)
commands/<name>.md   Rich Claude Code slash commands (full workflow, argument-hint, Gemini model selection, etc.)
skills/<name>/SKILL.md         Canonical Agent Skills — primary OpenCode entry point, also auto-discovered by Claude Code as plugin-namespaced skills
skills/<name>/COMMAND.opencode.md  Thin OpenCode command shim (overlay). Installer prefers this over commands/<name>.md when present, so OpenCode users get a small shim that delegates into SKILL.md while Claude users get the rich commands/<name>.md inline.
dist/                Built bundle (committed, self-contained)
```

## Core Behavior

- Relay core is backend-agnostic in `src/relay.ts` — `relay()` for batch, `relayStream()` for streaming, `reviewRelay()` for diff-scoped review, `relayBackground()` for quiet mode with job tracking
- Backend interface/registry in `src/backends/index.ts` — `run()` required, `runStream()` and `review()` optional, `capabilities` declares resume strategy and session ID requirements
- Shared `spawnCli()` async subprocess utility in `src/backends/index.ts` — used by all CLI backends (Antigravity, Codex, Claude, Gemini, OpenCode) for non-blocking execution with timeout, signal forwarding, stderr draining, and spawn error handling. Throws `SpawnCliError` (extends `BackendError`) on non-zero exit, preserving stdout/stderr/exitCode for callers that need partial output from failed runs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [freibergergarcia/phone-a-friend](https://github.com/freibergergarcia/phone-a-friend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
