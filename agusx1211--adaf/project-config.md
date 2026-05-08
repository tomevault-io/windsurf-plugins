---
trigger: always_on
description: Instructions and practices for AI agents working on this codebase.
---

# AGENTS.md

Instructions and practices for AI agents working on this codebase.

## Project Overview

ADAF (Autonomous Developer Agent Flow) is a Go CLI + React web UI that orchestrates AI coding agents (Claude, Codex, Gemini, Vibe, OpenCode, and arbitrary generic CLIs). It wraps these tools as child processes, parses their streaming output, records all I/O, and manages multi-agent collaboration through structured session handoffs, worktree isolation, and a persistent project store. A built-in web server with WebSocket support provides a real-time dashboard for monitoring and interacting with running sessions.

## Development Phase

This project is in early active development. There are no external users or deployments to maintain compatibility with. Storage formats, APIs, CLI flags, and data structures can change freely at any time — just wipe `.adaf/` and start fresh. Prefer the simplest, cleanest implementation over any backwards-compatibility concern. Aggressively refactor, rename, delete dead code, and restructure without hesitation. Never add migration logic, deprecation shims, or compatibility layers.

## Architecture

```
cmd/adaf/              Entry point
internal/
  agent/               Agent interface + per-tool implementations (claude, codex, vibe, gemini, opencode, generic)
  agentmeta/           Built-in metadata catalog (models, capabilities, reasoning levels)
  buildinfo/           Build metadata and version helpers
  cli/                 Cobra commands (~60 subcommands including daemon, web, sessions, stats, usage, skills)
  config/              Global user config (~/.adaf/config.json)
  debug/               Runtime debug tools and diagnostics
  detect/              PATH scanning, version probing, dynamic model discovery
  events/              Typed messages for web UI / daemon communication (WebSocket event protocol)
  eventq/              Local event queue and dispatch
  hexid/               Stable identifiers for sessions and work items
  loop/                Single-agent loop controller (turn management, recording, callbacks)
  looprun/             Multi-step loop runtime
  orchestrator/        Sub-agent spawning with worktree isolation and concurrency limits
  prompt/              Context-aware prompt building
  pushover/            Pushover notification integration
  recording/           Session I/O recording (NDJSON)
  session/             Detachable session management
  stats/               Statistics extraction from recordings
  store/               File-based project store (.adaf/)
  stream/              NDJSON stream parsing and terminal display (parsers for claude, codex, gemini, opencode, vibe)
  usage/               Provider usage tracking and rate-limit monitoring
  webserver/           Embedded web server with REST API, WebSocket, PTY, and TLS support
  worktree/            Git worktree lifecycle
web/                   React frontend (esbuild, compiled into webserver/static/)
  src/components/      UI components (feed, detail, views, project, common, layout, tree, session, loop)
e2e/                   Playwright end-to-end tests for the web UI
```

Key data flow: **CLI command -> Loop -> Agent.Run() -> child process (exec) -> stream parser -> recorder/event sink -> WebSocket -> web UI**

All agent integrations are CLI wrappers via `os/exec`. There are zero Go library dependencies on any agent SDK.

## Reference Repositories

The `./references/` directory contains cloned source code of the external agent CLIs that this project integrates with. **Always consult these when working on agent integrations** instead of guessing CLI flags, output formats, stream protocols, or model identifiers.

### Setup

```bash
mkdir -p references
git clone https://github.com/anthropics/claude-code references/claude-code
git clone https://github.com/openai/codex references/codex
git clone https://github.com/mistralai/mistral-vibe references/vibe
git clone https://github.com/google-gemini/gemini-cli references/gemini-cli
```

### When to consult references

| Area | Reference | What to verify |
|------|-----------|----------------|
| `internal/agent/claude.go` | `references/claude-code/` | CLI flags, `--output-format stream-json` schema, model IDs |
| `internal/agent/codex.go` | `references/codex/` | `exec` subcommand, `--json`, `--dangerously-bypass-approvals-and-sandbox`, model slugs |
| `internal/agent/vibe.go` | `references/vibe/` | `-p` flag, config.toml format, model aliases |
| `internal/agent/gemini.go` | `references/gemini-cli/` | `-p` flag, `--output-format stream-json`, `-y` auto-approve |
| `internal/detect/detect.go` | All | Version flags, install paths, config file locations, model discovery |
| `internal/stream/` | All | NDJSON event types, content block schemas, agent-specific stream formats |
| `internal/agentmeta/catalog.go` | All | Supported models, capabilities, defaults |

**Do not guess.** If a CLI flag, output format, or model name is in question, look it up in the reference source. If something isn't working as expected, `git -C references/<repo> pull` and verify against the latest.

The `references/` directory is git-ignored. Never commit files from it.

## Testing Philosophy


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Agusx1211/adaf](https://github.com/Agusx1211/adaf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
