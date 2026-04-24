---
trigger: always_on
description: > **Effective agent context for the Lemon AI assistant platform.**
---

# Lemon Agent Guide

> **Effective agent context for the Lemon AI assistant platform.**  
> Lemon is a local-first assistant and coding agent system with a multi-engine architecture supporting Claude, Codex, OpenCode, Pi, and native Lemon engines.

---

## Quick Navigation

| If you want to... | Look in... |
|-------------------|------------|
| Add/modify AI provider support | `apps/ai/` |
| Work on AI runtime auth facade boundary | `apps/lemon_ai_runtime/` |
| Work on coding tools or session management | `apps/coding_agent/` |
| Modify Telegram/Discord channel adapters | `apps/lemon_channels/` |
| Modify SMS/voice transports | `apps/lemon_gateway/` |
| Add new messaging channel adapters (X, XMTP, etc.) | `apps/lemon_channels/` |
| Work on agent routing or message flow | `apps/lemon_router/` |
| Build HTTP/WebSocket API features | `apps/lemon_control_plane/` |
| Manage configuration, secrets, or storage | `apps/lemon_core/` |
| Build reusable simulation harnesses | `apps/lemon_sim/` |
| Work with CLI runners/subagent spawning | `apps/agent_core/` |
| Create or modify skills | `apps/lemon_skills/` |
| Build cron jobs or automation | `apps/lemon_automation/` |
| Manage long-running external processes | `apps/lemon_services/` |
| Work on the web UI | `apps/lemon_web/` |
| Debug coding agent via RPC | `apps/coding_agent_ui/` |
| Market data ingestion | `apps/market_intel/` |
| Browser automation via CDP/Playwright | `clients/lemon-browser-node/` |

---

## Parallel Work & Git Worktrees

When working on multiple tasks in parallel (either as the same agent or multiple agents), **use git worktrees to avoid file editing conflicts**.
Store all worktrees under `.worktrees/` in the repository root.

### Workflow:

1. **Create a worktree for each parallel task:**
   ```bash
   mkdir -p .worktrees
   git worktree add .worktrees/task-name -b task-name
   cd .worktrees/task-name
   ```

2. **Work in isolation** — Each worktree is an independent working directory backed by the same repo:
   ```bash
   git status
   ```

3. **Clean up when complete** — After the branch is merged/closed, remove the worktree:
   ```bash
   cd /path/to/main/lemon
   git worktree remove .worktrees/task-name
   git branch -d task-name
   ```

### Why git Worktrees?

- **No file editing conflicts** — Multiple agents can edit different files simultaneously without stepping on each other
- **Clean build contexts** — Each worktree maintains separate `_build/` and `deps/` as needed
- **Easy cleanup** — Remove worktrees when done without affecting the main checkout

### Golden Rule:

> **Never have multiple agents editing the same working directory simultaneously.** Always use separate worktrees for parallel tasks.

---

## Agent Team Composition

When spawning agents for parallel work, **match the agent tier to the task complexity**. Don't use Opus for investigation or Sonnet for architectural decisions.

### Role Model

| Role | Internal Model | External Model | Typical Tasks |
|------|---------------|----------------|---------------|
| Junior/Mid Dev | Sonnet | Kimi | Investigation, plan file creation, test running, config cleanup, doc updates, dependency audits, simple refactors |
| Senior Dev | Opus | — | Complex refactoring, architectural extraction, correctness-critical code, multi-module decomposition |
| Staff Engineer | Codex (MCP) | — | Plan ownership/review, architectural oversight, cross-cutting design decisions, final validation |

### Guidelines

- **Default to the lowest tier that can do the job** — Use Sonnet for exploration and investigation. Only escalate to Opus when the task involves complex logic, cross-module refactoring, or correctness-critical code.
- **Codex owns plans** — Matches the existing `owner: codex` / `reviewer: codex` convention in the planning system. Codex reviews architectural decisions and validates decomposition strategies.
- **Escalation pattern**: Sonnet investigates → Opus implements → Codex reviews. Not every task needs all three tiers.
- **Kimi for external/security**: Security audits, pre-push hooks, and external review tasks (already established in the pre-push hook workflow).
- **Planning metadata**: `owner:` and `reviewer:` fields in plan YAML front matter should reference these roles (e.g., `owner: codex`, `reviewer: codex`).

### Spawning Examples

```bash
# Sonnet for investigation (junior/mid)
# Use model: "sonnet" in Task tool or --model sonnet in CLI

# Opus for complex implementation (senior)
# Use model: "opus" in Task tool or default CLI model

# Codex for plan review (staff)
# Use mcp__codex__codex tool with architectural review prompt

# Kimi for security audit (external)
# Use kimi CLI runner for pre-push or security review
```

---

## Documentation Contract ⚠️

> **Work is not complete until it is adequately documented.**

**Any code change must be accompanied by updates to all relevant documentation.** This is non-negotiable. Outdated documentation is technical debt that compounds and confuses future developers (including yourself).

### When You Modify Code, You MUST:

1. **Update `AGENTS.md` files** — If you change architecture, patterns, dependencies, or behaviors described in any `AGENTS.md`, update it immediately.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [z80dev/lemon](https://github.com/z80dev/lemon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
