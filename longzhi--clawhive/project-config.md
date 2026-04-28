---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Project Overview

Clawhive is a Rust-native single-binary (~14MB) multi-agent AI platform for deploying agents across messaging channels (Telegram, Discord, Slack, WhatsApp, iMessage). 13-crate workspace, edition 2021, Rust **1.92.0+**, version `0.1.0-alpha.*`.

## Agent Workflow

### Planning
- ≥3 steps or architecture decisions → enter plan mode first
- Execution deviates → stop immediately, re-plan. Never force through
- Plans must include verification steps, not just build steps
- Write detailed specs upfront to reduce ambiguity

### Subagents
- Use subagents liberally to keep main context clean
- Delegate research, exploration, parallel analysis to subagents
- Throw more compute at complex problems via subagents
- One subagent = one focused task

### Self-Optimization
- On any correction → update pattern to `tasks/lessons.md`
- Set rules for yourself to prevent repeat mistakes
- Iterate on lessons strictly until error rate drops
- Review `tasks/lessons.md` at session start

### Verification Before Completion
- Never mark done until functionality is proven working
- Compare behavior against main branch when needed
- Self-check: "Would a senior engineer approve this?"
- Run tests, check logs, prove correctness

### Elegance (Balanced)
- For non-trivial changes → pause and ask "Is there a more elegant way?"
- Hacky fix → rewrite with full context for an elegant solution
- Simple, clear fixes → skip this step, avoid over-engineering
- Cross-review your own work before delivery

### Bug Fixing
- On bug report → fix directly, no hand-holding needed
- Locate logs, errors, failing tests → complete the fix
- Zero context-switching required from user
- Proactively fix failing CI without being asked

### Task Management
1. Write execution plan with checkable items to `tasks/todo.md`
2. Verify plan before starting development
3. Mark items complete as you go
4. Add high-level summary for each step
5. Add retrospective section to `tasks/todo.md` on completion
6. On corrections → update `tasks/lessons.md`

### Core Principles
- **Minimal footprint**: smallest possible code changes
- **No shortcuts**: find root cause, no temp fixes, senior engineer standards

## Build / Test / Lint

```bash
# One-time setup (git hooks: fmt+clippy on commit, full check on push)
just install-hooks

# Full CI-equivalent quality gate (run before every PR)
just check
# Equivalent to:
#   1. cargo fmt --all -- --check
#   2. cargo clippy --workspace --all-targets -- -D warnings
#   3. cargo test --workspace

# Individual commands
just fmt                # cargo fmt --all (auto-format)
just fmt-check          # cargo fmt --all -- --check
just clippy             # cargo clippy --workspace --all-targets -- -D warnings
just test               # cargo test --workspace
cargo build --release   # release binary

# Run a single test (by name substring)
cargo test -p clawhive-core -- policy::tests::check_exec -v

# Run all tests in one crate
cargo test -p clawhive-core

# Run tests matching a pattern
cargo test -p clawhive-scheduler -- integration

# Frontend (in web/ directory, use bun not npm)
cd web && bun install && bun run build
cd web && bun run dev   # dev server with proxy to localhost:3001

# Deploy to Mac Studio (pushes dev, builds remotely, restarts daemon)
./deploy.sh

# Version release (tags on main, not dev)
just release patch  # or minor/major
```

CI runs 4 parallel jobs on `ubuntu-latest`: check, test, clippy, fmt. `RUSTFLAGS=-Dwarnings` is set globally in CI — **all warnings are errors**.

## Workspace Structure

```
crates/
├── clawhive-cli/        # CLI binary (clap) — the only bin crate
├── clawhive-core/       # Orchestrator, tools, policy, skills, config — most logic lives here
├── clawhive-memory/     # Memory system (file store, JSONL sessions, SQLite index, embedding)
├── clawhive-gateway/    # Gateway, agent routing, rate limiting, scheduled task listener
├── clawhive-bus/        # In-process event bus (pub/sub)
├── clawhive-provider/   # LLM provider trait + multi-provider adapters
├── clawhive-channels/   # Channel adapters (Telegram, Discord, Slack, WhatsApp, iMessage)
├── clawhive-auth/       # OAuth and API key auth
├── clawhive-scheduler/  # Cron-based task scheduling
├── clawhive-server/     # HTTP API server (axum) + embedded SPA
├── clawhive-schema/     # Shared DTOs (InboundMessage, OutboundMessage, BusMessage)
├── clawhive-runtime/    # Task executor abstraction
└── clawhive-tui/        # Terminal dashboard (ratatui)
```

Dependency flow (top → bottom):

```
clawhive-cli
  ├─ clawhive-tui
  ├─ clawhive-server
  ├─ clawhive-gateway
  │    ├─ clawhive-channels
  │    └─ clawhive-core
  │         ├─ clawhive-provider
  │         ├─ clawhive-memory
  │         ├─ clawhive-scheduler
  │         └─ clawhive-auth
  ├─ clawhive-bus
  ├─ clawhive-runtime
  └─ clawhive-schema
```

### Key Source Files

- `clawhive-core/src/orchestrator.rs` — ReAct reasoning loop, tool execution, sub-agent spawning
- `clawhive-core/src/persona.rs` — Agent identity construction and system prompt
- `clawhive-core/src/skill.rs` — Skill loading, permission checking from SKILL.md

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [longzhi/clawhive](https://github.com/longzhi/clawhive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
