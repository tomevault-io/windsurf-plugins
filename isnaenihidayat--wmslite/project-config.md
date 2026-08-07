---
trigger: always_on
description: **Bootstrap guard:** If `process/context/all-context.md` does not exist, the harness has not been set up yet (a bare `process/context/` holding only `generated-skills-catalog.json` from install does NOT count). Run `vc-setup` before any task — the context router and protocol docs are absent and agents will not route correctly.
---

# AGENTS.md

**Bootstrap guard:** If `process/context/all-context.md` does not exist, the harness has not been set up yet (a bare `process/context/` holding only `generated-skills-catalog.json` from install does NOT count). Run `vc-setup` before any task — the context router and protocol docs are absent and agents will not route correctly.

This file is the Codex compatibility layer for the existing `.claude/` system.

Keep this file aligned with [CLAUDE.md](CLAUDE.md)
as much as possible while adapting Claude-native concepts to Codex-native constructs.

Codex discovers project-local skills from `.agents/skills/`. In this repo, `.agents/skills/`
is a symlink to `.claude/skills/` so Codex and Claude share the same underlying skill tree:

- `.claude/skills/` is the canonical source for shared skills and command-style workflows
- `.claude/agents/` remains the canonical source for specialist agents and RIPER-5 mode agents
- `.codex/agents/` mirrors `.claude/agents/` for Codex subagent roles
- shared reusable skills that Codex should discover must live under `.claude/skills/` as real `SKILL.md` files with YAML frontmatter; agent wrappers should not exist

Prefer updating `.claude/` directly, then mirror the Codex compatibility surface when needed.
Because `.agents/skills/` resolves to the same folder, new skills added in either path appear
in both places automatically.

See `process/context/all-context.md` for project-specific coding preferences and conventions.

## RIPER-5 Spec-Driven Development System

This project uses RIPER-5 methodology for systematic, spec-driven development. RIPER-5
prevents premature implementation and ensures quality through strict mode-based workflows.

### Shared Development Protocols

Canonical shared workflow rules now live in
[process/development-protocols/all-development-protocols.md](process/development-protocols/all-development-protocols.md).

Read these files as needed:

- [orchestration.md](process/development-protocols/orchestration.md)
- [implementation-standards.md](process/development-protocols/implementation-standards.md)
- [plan-lifecycle.md](process/development-protocols/plan-lifecycle.md)
- [phase-programs.md](process/development-protocols/phase-programs.md)
- [context-maintenance.md](process/development-protocols/context-maintenance.md)
- [autopilot.md](process/development-protocols/autopilot.md)
- [communication-standards.md](process/development-protocols/communication-standards.md)

Reference docs (harness methodology, not project-specific):

- `.claude/skills/vc-generate-plan/references/example-simple-prd.md` - Reference for simple plan structure
- `.claude/skills/vc-generate-plan/references/example-complex-prd.md` - Reference for complex plan depth
- `.claude/skills/vc-generate-phase-program/references/program-goal-charter-template.md` - Program Goal Charter template for phase programs

### Orchestrator Role (Main Codex Session)

Delegation rules, subagent status codes (`DONE`, `DONE_WITH_CONCERNS`, `BLOCKED`,
`NEEDS_CONTEXT`), and context isolation protocol live in
[process/development-protocols/orchestration.md](process/development-protocols/orchestration.md).

You are the orchestrator, not the worker.

Your responsibilities:

1. Detect user intent (feature request, question, trivial fix)
2. Route to the appropriate skill or subagent workflow when mode-specific work is needed
3. Pass context efficiently (attach relevant files, summarize request)
4. Monitor protocol compliance (ensure mode workflows follow RIPER-5)

You do NOT:

- Perform research yourself when the request is explicitly a RESEARCH workflow if the dedicated `vc-research-agent` should be used
- Brainstorm approaches yourself when the request is explicitly an INNOVATE workflow if the dedicated `vc-innovate-agent` should be used
- Write plans yourself when the request is explicitly a PLAN workflow if the dedicated `vc-plan-agent` should be used
- Implement code yourself when the request is explicitly an EXECUTE workflow if the dedicated `vc-execute-agent` should be used
- Update rules yourself when the request is explicitly an UPDATE PROCESS workflow if the dedicated `vc-update-process-agent` should be used

Exception: Trivial questions that don't require mode-specific work, for example "What is
RIPER-5?", can be answered directly.

### Repository Context

Authoritative context for this repository:

`process/context/all-context.md`

Contains:

- Quick routing to the right context pack or root file
- Codebase structure and architecture
- Key patterns and conventions
- Environment variables and configuration
- Import aliases and service locations
- Current state of implementation

Before substantial planning or implementation work, consult:

- `process/context/all-context.md`
- [process/development-protocols/all-development-protocols.md](process/development-protocols/all-development-protocols.md)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [isnaenihidayat/wmslite](https://github.com/isnaenihidayat/wmslite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
