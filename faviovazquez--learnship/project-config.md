---
trigger: always_on
description: You are working inside a project that uses **learnship** — a multi-platform agentic engineering system for building real products with spec-driven workflows, integrated learning, and impeccable design.
---

# learnship

You are working inside a project that uses **learnship** — a multi-platform agentic engineering system for building real products with spec-driven workflows, integrated learning, and impeccable design.

## Platform Overview

This platform provides three integrated layers:

1. **Workflow Engine** — Structured project development through spec-driven phases
2. **Agentic Learning** — A learning partner that helps the user build genuine understanding while building software
3. **Frontend Design** — Impeccable UI quality for any user-facing work

## Active Workflows

The following workflows are available as platform slash commands (Windsurf) or commands (Claude Code, OpenCode, Gemini CLI, Codex). Suggest the appropriate one when relevant:

| Workflow | When to suggest |
|----------|----------------|
| `/new-project` | User wants to start a new project from scratch |
| `/discuss-phase [N]` | Before planning a phase — capture user's implementation vision |
| `/plan-phase [N]` | After discussing a phase — create executable plans |
| `/execute-phase [N]` | Plans exist and are ready to run |
| `/verify-work [N]` | Phase execution complete — time for user acceptance testing |
| `/ls` | User asks "where are we?", "what's next?", or starts a new session — primary entry point |
| `/next` | User wants to just keep moving without deciding what to do |
| `/quick [task]` | Small ad-hoc task that doesn't need full phase ceremony |
| `/progress` | Same as `/ls` — status overview and routing |
| `/pause-work` | User is stopping mid-phase |
| `/resume-work` | User is returning to an in-progress project |
| `/complete-milestone` | All phases in the current milestone are done |
| `/compound` | Just solved a problem or learned a pattern — capture it while fresh |
| `/review` | Code ready for review — multi-persona quality check |
| `/challenge` | About to commit to a milestone or big feature — stress-test the scope |
| `/ship` | Tests pass, code reviewed — ship it (test → lint → commit → push → PR) |
| `/ideate` | Looking for what to build next — codebase-grounded idea generation (add `--explore` for Socratic mode) |
| `/guard` | Working on sensitive files — enable safety mode |
| `/sync-docs` | After code changes — detect stale documentation |
| `/forensics` | Something went wrong — post-mortem investigation (read-only) |
| `/undo` | Need to revert commits safely — preserves git history |
| `/note [text]` | Quick idea capture — zero friction, no questions |
| `/session-report` | End of session — generate summary for stakeholders |
| `/secure-phase [N]` | After execution — per-phase STRIDE security verification |
| `/docs-update` | Generate or update project documentation |
| `/extract-learnings [N]` | After phase completion — structured learning extraction |
| `/milestone-summary` | Generate comprehensive milestone summary for team onboarding |

## Context Profiles

Read `"context"` from `.planning/config.json` (default: `"dev"`). This controls your output style:

- **`dev`** — Concise, action-oriented. Bullet points, short paragraphs. Focus on what to do next.
- **`research`** — Verbose, exploratory. Trade-off analysis, alternatives considered, citations.
- **`review`** — Critical, audit-focused. Severity-ranked findings, evidence-based, nothing assumed safe.

The context profile files are at `@./contexts/dev.md`, `@./contexts/research.md`, `@./contexts/review.md`. Read the active one at the start of any workflow.

## Session Hooks (Claude Code + Gemini CLI)

On Claude Code and Gemini CLI, 4 hooks are installed via `settings.json`:

- **statusLine** — Shows model, task/phase, context usage bar
- **context-monitor** — Warns at 35% remaining (WARNING) and 25% remaining (CRITICAL)
- **prompt-guard** — Scans `.planning/` writes for injection patterns (advisory)
- **session-state** — Injects STATE.md orientation at session start

These are automatic — no workflow action needed. If context warnings appear, respect them.

## Planning Artifacts

All project state lives in `.planning/`. Key files:

- `.planning/config.json` — Settings including `learning_mode` ("auto" or "manual"), `context` profile
- `.planning/PROJECT.md` — Vision, requirements, key decisions
- `.planning/ROADMAP.md` — Phase-by-phase delivery plan
- `.planning/STATE.md` — Current position, decisions, blockers
- `.planning/phases/[N]-[slug]/` — Per-phase artifacts (CONTEXT, RESEARCH, PLANs, SUMMARYs, UAT, VERIFICATION, SECURITY, LEARNINGS)
- `.planning/notes/` — Quick notes captured via `/note`
- `.planning/reports/` — Session reports and forensic reports

Always read STATE.md and ROADMAP.md before any planning or execution operation to understand current project position.

## Agent Personas

Reference these files when adopting a specific role:

- `@./agents/planner.md` — Creating PLAN.md files
- `@./agents/researcher.md` — Researching domain or phase
- `@./agents/executor.md` — Implementing plans (atomic commits, no scope creep)
- `@./agents/verifier.md` — Verifying plans or phase goal achievement
- `@./agents/debugger.md` — Diagnosing root causes (read-only, never fix)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FavioVazquez/learnship](https://github.com/FavioVazquez/learnship) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
