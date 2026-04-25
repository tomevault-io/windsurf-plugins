---
trigger: always_on
description: Meet the future of IDEs. Swarmify turns your editor into an IAE — orchestrate Claude, Codex, Gemini, and Cursor in parallel with full visibility.
---

# Swarmify - The Integrated Agents Environment

Meet the future of IDEs. Swarmify turns your editor into an IAE — orchestrate Claude, Codex, Gemini, and Cursor in parallel with full visibility.

## Architecture

```
+------------------+     +------------------+     +------------------+
|   Claude Code    |     |      Codex       |     |      Gemini      |
+--------+---------+     +--------+---------+     +--------+---------+
         |                        |                        |
         +------------------------+------------------------+
                                  |
                    +-------------+-------------+
                    |     Swarmify MCP Server   |
                    |   (Agent Orchestration)   |
                    +-------------+-------------+
                                  |
                    +-------------+-------------+
                    |    VS Code / Cursor IDE   |
                    |  (Extension: Editor Tabs) |
                    +---------------------------+
```

## Repository Structure

| Directory | Package | Purpose |
|-----------|---------|---------|
| `agents-mcp/` | @swarmify/agents-mcp | MCP server for spawning agents |
| `extension/` | swarm-ext | VS Code/Cursor extension |
| `prompts/` | - | Slash commands for all agents |

## Core Concepts

### Agent Types

| Agent | CLI | Strength | Best For |
|-------|-----|----------|----------|
| Claude | `claude` | Maximum capability, research, exploration | Complex research, open-ended exploration |
| Codex | `codex` | Fast, cheap | Self-contained features |
| Gemini | `gemini` | Complex multi-system features | Architectural changes |
| Cursor | `cursor-agent` | Debugging, bug fixes | Tracing through codebases |

### Agent Modes

| Mode | File Access | Auto-loops? | Use Case |
|------|-------------|-------------|----------|
| `plan` | Read-only | No | Research, exploration, code review |
| `edit` | Read + Write | No | Implementation, refactoring, fixes |
| `ralph` | Full yolo | Yes | DEPRECATED in agents-mcp 0.3.0, removed in 0.4.0 |

Default is `plan` for safety.

### Agent Orchestration

Agents can spawn other agents via MCP tools:
- Claude can spawn Codex for a quick fix while continuing analysis
- Orchestrator assigns tasks, prevents conflicts by splitting work by file
- Subagents run asynchronously - orchestrator polls for progress

## Hard Rules

CROSSING ANY OF THESE HARD LINES EVEN ONCE WILL RESULT IN IMMEDIATE AND PERMANENT TERMINATION OF THE PROFESSIONAL RELATIONSHIP BETWEEN THE USER AND CLAUDE.

### 1. NO EMOJIS, ICONS, OR DECORATIVE SYMBOLS
Not in code. Not in comments. Not in commits. Not in UI text. Not in any file. No checkmarks. No stars. No sparkles. No visual flair of any kind.
**Exception:** User explicitly confirms 3+ times with phrases like "YES I REALLY WANT EMOJIS".

### 2. NO MOCKING IN TESTS
All tests must use real services. No mocks. No stubs. No fakes. End-to-end testing with real execution.

### 3. NO ENV VARS FOR USER CREDENTIALS OR INFRASTRUCTURE
Never use environment variables for:
- User tokens, OAuth keys, personal data
- Infrastructure URLs (proxy URLs, API endpoints) - HARDCODE THESE
**Only OK:** Server-side API keys in .env files
For user credentials: use Keychain, encrypted config files, or secure storage.

### 4. NO TESTS IN /TMP
Tests belong in the codebase, not /tmp. Never write tests, test scripts, or verification code to /tmp. If you need to verify something works, write a proper test that can be rerun.

### 5. NO BACKGROUND SHELLS
Never use `run_in_background: true` with the Bash tool. Background shells accumulate and crash. All commands must run in the foreground.

### 6. NO TOASTS
Never use toast notifications in UI. Operations must either work silently (one-click, no feedback) or show inline errors (red text near the action). Success is assumed.
**Exception:** Critical system errors that require immediate user attention.

## Development Defaults

- Package manager: bun (not npm/yarn/pnpm)
- TypeScript only (no plain JS)
- Python: loguru for logging, built-in type hints
- Env files: .env.dev and .env.prod (not .env.example)
- Async-first for disk/network IO
- Minimal libraries
- Extend existing code over writing new

## Code Style

- No comments unless explicitly requested
- Test fixtures in `testdata/` subdirectory (same directory as test file)
- One test file per concern
- Go tests MUST use testify: `require.NoError(t, err)` not `if err != nil { t.Fatal(err) }`

## Tech Stack

- Frontend: Node v24, Next.js, Bun, React, Tailwind, zustand, lucide-react
- Backend: Python 3.12, FastAPI, uv, pydantic, loguru, Supabase/Postgres
- MCP: @modelcontextprotocol/sdk

## Architecture Patterns

### Agent Spawning (Swarm MCP)
Use Swarm MCP tools: `spawn`, `status`, `stop`, `tasks`
- Spawn agents FIRST before other work (parallelism)
- Agents execute, you architect - delegate with specific context
- Do NOT use built-in Claude Code Task tool when Swarm agents are requested

### Session Activity Parsing
Live activity extraction from agent session files to show what agents are doing in Dashboard.

**Agent formats** (critical - each agent logs differently):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [muqsitnawaz/swarmify](https://github.com/muqsitnawaz/swarmify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
