---
trigger: always_on
description: Universal orchestration engine for AI-powered development. Single Rust binary (`forge`) that decomposes specs into tasks, dispatches them to Claude/Codex/Gemini, and reconciles results.
---

# CLAUDE.md — Forge Orchestrator

Universal orchestration engine for AI-powered development. Single Rust binary (`forge`) that decomposes specs into tasks, dispatches them to Claude/Codex/Gemini, and reconciles results.

## Quick Reference

```bash
# Build
cargo build --release          # Optimized binary (4 MB, LTO + strip)
cp target/release/forge ~/.local/bin/forge-orca

# Test
cargo test                     # All 362 tests (340 unit + 10 CLI + 12 MCP)
cargo clippy -- -W clippy::all # Lint

# Run
cargo run -- init              # Initialize .forge/ in a project
cargo run -- plan --generate   # Generate plan from SPEC.md
cargo run -- dashboard --pty   # TUI dashboard with native agent TUIs
cargo run -- mcp               # Start MCP server (stdio JSON-RPC 2.0)
cargo run -- status            # Task board + event log
cargo run -- uat               # Interactive UAT TUI
```

**IMPORTANT:** Stop all `forge-orca` processes before `cp` — "Text file busy" error otherwise.
**GPG:** Use `--no-gpg-sign` for commits in this repo.

## Architecture

```
src/
├── main.rs              # Entry: tokio + clap CLI (13 subcommands)
├── cli/                 # Command handlers (init, plan, run, start, dashboard, status,
│                        #   sync, mcp, config, verify, uat)
├── core/                # Deterministic engine
│   ├── state.rs         #   .forge/state.json — project metadata, auth modes, locks
│   ├── task.rs          #   Task lifecycle — Pending→InProgress→Completed/Failed
│   ├── event.rs         #   .forge/events.jsonl — append-only audit trail
│   ├── governance.rs    #   5-dimension health score + drift detection
│   ├── knowledge.rs     #   .forge/knowledge/ — research/decisions/learnings/patterns
│   ├── finding.rs       #   UAT findings with severity classification
│   └── plan.rs          #   .forge/plan.md + plan.yaml
├── adapters/            # AI tool adapters (Claude, Codex, Gemini)
│   ├── mod.rs           #   ToolAdapter trait (render_config, build_command, ready_pattern)
│   ├── claude.rs        #   Writes CLAUDE.md, smart --allowedTools by task type
│   ├── codex.rs         #   Writes AGENTS.md, codex --task-mode
│   └── gemini.rs        #   Writes GEMINI.md, gemini CLI
├── mcp/                 # MCP server (stdio transport, 10 tools)
│   ├── server.rs        #   JSON-RPC 2.0 dispatcher
│   ├── tools.rs         #   Tool implementations
│   └── protocol.rs      #   Type definitions
├── tui/                 # Terminal UI (ratatui + crossterm)
│   ├── app.rs           #   Dashboard state + execution engine (largest module)
│   ├── ui.rs            #   Rendering (task board, agent panes, event log)
│   ├── pty_session.rs   #   PTY allocation + vt100 terminal emulator
│   ├── uat_app.rs       #   UAT TUI state machine
│   └── uat_ui.rs        #   UAT TUI rendering
├── brain/               # Pluggable LLM backends
│   ├── mod.rs           #   ForgeBrain trait
│   ├── rule_based.rs    #   Free tier (heuristics, no LLM)
│   └── openai.rs        #   GPT-4o/o3/o4-mini via OpenAI API
└── detect/              # Tool detection (scan PATH for claude, codex, gemini)
```

## MCP Tools (11)

forge-orchestrator exposes these tools via `forge mcp` (stdio JSON-RPC 2.0):

| Tool | Purpose |
|------|---------|
| `forge_get_tasks` | List tasks (filter by status) |
| `forge_claim_task` | Claim task for an agent (locks files) |
| `forge_complete_task` | Mark task done (unlocks files, logs event) |
| `forge_get_state` | Full orchestration state |
| `forge_get_plan` | Master plan (plan.md) |
| `forge_capture_knowledge` | Capture learning (auto-classifies) |
| `forge_get_knowledge` | Query knowledge base |
| `forge_check_drift` | Compare work vs. vision (SPEC.md) |
| `forge_get_health` | Comprehensive governance health check |
| `forge_set_project` | Switch active project |
| `forge_get_events` | Query event history (count, task_id, event_type filters) |

## Task Model

**Phases:** Build → Verify → Fix (auto-transitions in dashboard)
**Statuses:** Pending → Assigned → InProgress → Completed / Failed / Blocked
**ID Prefixes:** T-xxx (build), V-xxx (verify), F-xxx (fix)

Tasks have: `assigned_to` (agent), `locked_files` (exclusive access), `depends_on` (blocking), `task_type` (design/implement/review/test/document), `acceptance_criteria`.

## Adapter Pattern

Each AI tool has an adapter implementing `ToolAdapter`:

| Method | Purpose |
|--------|---------|
| `render_config()` | Write context file (CLAUDE.md, AGENTS.md, GEMINI.md) |
| `build_command()` | CLI command for headless execution |
| `build_command_interactive()` | CLI command for PTY/TUI mode |
| `initial_input()` | Text typed into TUI after startup |
| `ready_pattern()` | Pattern indicating TUI ready for input |

**Ready patterns:** Claude=`"to cycle"`, Codex=`"help you with"`, Gemini=`"Type your message"`

### Stargate Auto-Approve Contract (MANDATORY)

Every adapter's `build_command_interactive()` MUST pass the tool's auto-approve flag unconditionally in Stargate mode. Agents inside dashboard PTY panes run unattended — they cannot wait for human approval prompts.

| Tool | Auto-Approve Flag | Status |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nxtg-ai/forge-orchestrator](https://github.com/nxtg-ai/forge-orchestrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
