---
trigger: always_on
description: Forge is a Claude Code CLI plugin that provides an autonomous, spec-driven development loop.
---

# Forge — Autonomous Agent Coding System

## What This Is
Forge is a Claude Code CLI plugin that provides an autonomous, spec-driven development loop.
Three commands: `/forge brainstorm`, `/forge plan`, `/forge execute` — they chain together
to take an idea from concept to working code with minimal human intervention.

## Project Structure
```
forge/
├── .claude-plugin/plugin.json     — Plugin manifest
├── commands/                      — Slash commands (/forge brainstorm, plan, execute, etc.)
├── skills/                        — Procedural workflows + cross-cutting skills (guardrails, graphify, design-system)
├── agents/                        — Specialized subagents (speccer, planner, executor, reviewer, verifier, researcher)
├── hooks/                         — Stop hook (loop engine), token monitor (PostToolUse)
├── scripts/                       — JS utility (forge-tools.cjs) + bash helpers
├── templates/                     — Output file templates (spec, plan, state, summary)
├── references/                    — Reference docs (token profiles, backprop patterns, multi-repo, etc.)
└── docs/superpowers/specs/        — Design specs for this project
```

## Workflow Enforcement (CRITICAL)

The Forge workflow is strictly sequential: **brainstorm -> plan -> execute**. This is enforced at multiple levels:

1. **Spec approval gate**: Only the brainstorming skill writes `status: approved` specs, and only after explicit user approval of an approach. The execute command validates this.
2. **Frontier requirement**: `/forge execute` validates that every approved spec has a corresponding frontier file from `/forge plan`.
3. **Programmatic validation**: `forge-tools.cjs setup-state` runs `validateWorkflowPrerequisites()` which checks spec approval status and frontier existence before allowing execution.
4. **State machine phases**: `brainstorming` and `planning` are formal phases in the state machine (see `references/state-machine.md`).

**Never skip brainstorming.** Even if the user's request seems clear, the interactive Q&A surfaces hidden assumptions, the approach proposals prevent over-engineering, and the approved spec provides verifiable acceptance criteria for the reviewer and verifier.

## Architecture
- **Lean plugin** — installable via `claude plugin install forge`, no npm dependency for users
- **Smart loop** — Stop hook reads state and routes to the correct next action (not dumb re-feed)
- **Three-layer loop** — Outer (phase progression) → Middle (task progression) → Inner (quality iteration)
- **Adaptive depth** — Auto-detects complexity, scales ceremony (quick/standard/thorough), user can override
- **Context resets** — At 60% context usage, saves handoff snapshot and starts fresh session
- **Token budget** — PostToolUse hook tracks usage, auto-downgrades depth when budget runs low
- **Capability discovery** — Scans for user's MCP servers and skills, routes work to leverage them
- **Multi-repo** — Natively coordinates work across multiple repos (API-first ordering)
- **Backpropagation** — Traces runtime bugs back to specs, generates regression tests
- **Live TUI dashboard** — Opt-in visualization layer (`/forge watch` or `FORGE_TUI=1`) parses `claude --output-format stream-json` and renders a zero-dependency ANSI dashboard via `scripts/forge-tui.cjs`. Augments the bash runner; falls back to plain mode automatically on sentinel exit code 87

## Key Conventions
- All state lives in `.forge/` per-project (gitignored)
- Specs: `.forge/specs/spec-{domain}.md` with R-numbered requirements
- Plans: `.forge/plans/{spec}-frontier.md` with tiered task DAGs
- State: `.forge/state.md` tracks current position, decisions, progress
- Token ledger: `.forge/token-ledger.json` tracks cumulative usage
- Atomic commits per task with descriptive messages
- Circuit breakers prevent infinite loops (3x fail → debug mode, 3x debug → human)

## Platform
- **Target**: Windows (Claude Code runs in WSL, but plugin should be cross-platform compatible)
- **Shell scripts**: Use `#!/usr/bin/env bash` for portability
- **JS utility**: Node.js (forge-tools.cjs) — CommonJS for broad compatibility
- **Path handling**: Always use forward slashes in JS, handle Windows paths in bash scripts
- **No native dependencies** — pure JS + bash, no compilation step

## Tech Stack
- Plugin format: Claude Code plugin spec (plugin.json, commands/, skills/, agents/, hooks/)
- Scripting: Node.js (CommonJS) for forge-tools.cjs, Bash for hooks
- State: Markdown files + JSON (no database)
- No build step, no bundler, no framework

## Development Workflow
- Design specs in `docs/superpowers/specs/`
- Test locally: `claude --plugin-dir /home/lucasduys/forge`
- Reload without restart: `/reload-plugins`
- Keep scripts POSIX-compatible where possible for cross-platform

## Code Style
- JS: CommonJS (`require`/`module.exports`), no TypeScript (keep it simple for contributors)
- Markdown: YAML frontmatter for metadata, consistent heading hierarchy
- Agent prompts: Clear role, explicit constraints, output format specified
- Bash: `set -euo pipefail`, quote all variables, use `${CLAUDE_PLUGIN_ROOT}` for paths

## What NOT To Do
- Don't add npm dependencies — this must be zero-install for users

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LucasDuys/forge](https://github.com/LucasDuys/forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
