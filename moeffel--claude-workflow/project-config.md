---
trigger: always_on
description: Standard development workflow for Claude. Works everywhere: CLI, Desktop, IDE, and Cloud.
---

# claude-workflow

Standard development workflow for Claude. Works everywhere: CLI, Desktop, IDE, and Cloud.

On **CLI/Desktop**: Run `bash setup.sh` once after cloning to install plugins (superpowers, ECC, codex, etc.). Skills referenced below will activate automatically.

On **Claude Cloud**: No setup needed. This file IS the workflow — follow the instructions directly. Plugins, hooks, and MCP servers are declared in `.claude/settings.json` and `.mcp.json` (committed to the repo) and activate automatically on Cloud.

> **100% Cloud-ready**: MemPalace auto-installs on first session via SessionStart hook. All agents, skills, hooks, and MCP config are repo-committed. No local setup required.

---

## Standard Workflow

Follow this workflow for ALL implementation tasks. Each step is mandatory unless explicitly skipped by the user. If a skill mentioned below is available, invoke it. If not, follow the written instructions directly.

### Session Start (MANDATORY — run at the beginning of EVERY session)

Before any work begins, load project memory to establish continuity:

1. **Load MemPalace context** — if MemPalace MCP is available, run `wake-up` or search for this project's wing to load critical facts (~170 tokens).
2. **Check for open work** — search memory for: unfinished tasks, blockers from last session, pending decisions.
3. **Read recent git log** — `git log --oneline -10` to see what happened since last session.
4. **Read active plan** — check `docs/superpowers/plans/` for any in-progress plan and its current phase.
5. **Brief the user** — summarize what you found: where we left off, what's next, any open questions.

If MemPalace is not available, fall back to git log + plan docs + CLAUDE.md context only.

> This step ensures no context is lost between sessions. Skip ONLY if the user explicitly says "fresh start".

---

### Step 0: Research & Reuse

**Before writing ANY new code**, spawn the **researcher** agent or search manually:

1. Search GitHub for existing implementations (`gh search code`, `gh search repos`)
2. Check library docs for API behavior and patterns (Context7 or official docs)
3. Search package registries (npm, PyPI, crates.io) — prefer battle-tested libraries over hand-rolled code
4. Search the web for broader context only if steps 1-3 are insufficient
5. Look for open-source projects that solve 80%+ of the problem

Prefer adopting a proven approach over writing from scratch. **The researcher agent is ALWAYS the first to run** — no exceptions.

> CLI skill chain: `search-first` → `docs` → `deep-research` → `exa-search`

### Step 1: Brainstorm & Spec

**Mandatory before any creative or design work.** Do not jump to implementation.

1. Generate multiple approaches (at least 3). Consider trade-offs for each.
2. **Expand into a full spec** using `spec-expander` skill:
   - Interview the user to fill gaps (5-8 focused questions)
   - Write structured spec: Problem, Users, Requirements (MUST/SHOULD/COULD), Acceptance Criteria, Out of Scope
   - Save to `docs/specs/YYYY-MM-DD-[name].md`
3. **Review the spec** using `spec-reviewer` skill:
   - Score 7 dimensions (Clarity, Testability, Scope, Feasibility, Consistency, Security, Completeness)
   - Adversarial challenge on every MUST requirement
   - Verdict: KILL / FIX / SHIP — only proceed to planning on SHIP
4. If available, get a cross-model review via `/codex:review` on the spec file for an independent perspective.

> CLI skills: `superpowers:brainstorming` → `spec-expander` → `spec-reviewer`
> Optional plugin: `spec-kit` (athola/claude-night-market) for additional spec tooling

### Step 2: Plan

1. Create a structured implementation plan before writing code:
   - Break into phases (each independently testable)
   - Identify dependencies between phases
   - List risks and mitigation strategies
   - Estimate complexity per phase
2. Write the plan to `docs/superpowers/plans/YYYY-MM-DD-name.md`
3. Get user confirmation before proceeding to implementation

> CLI skills: `superpowers:writing-plans` + **planner** agent

### Step 3: Implement

1. Execute the plan phase by phase. Do not skip ahead.
2. For each phase, follow TDD strictly:
   - **RED**: Write the test first. Run it. It MUST fail.
   - **GREEN**: Write the minimal code to make the test pass.
   - **IMPROVE**: Refactor while keeping tests green.
3. Target 80%+ test coverage.
4. Use parallel agents/subagents for independent tasks within a phase.
5. After each file edit, review your own change for obvious issues before moving on.

> CLI skills: `superpowers:executing-plans` + `superpowers:subagent-driven-development` + `tdd` + `quality-gate`

### Step 4: Review

1. Review all code changes before considering work done:
   - **Correctness**: Does it do what was asked? Edge cases handled?
   - **Security**: No hardcoded secrets? Inputs validated? Queries parameterized? Auth checked?
   - **Quality**: Functions < 50 lines? Files < 800 lines? No deep nesting? Immutable patterns?
   - **Tests**: Do tests cover the happy path AND error cases?
2. For security-sensitive code (auth, payments, user input, DB, crypto): do an explicit security review.
3. For database changes: review schema design, query performance, migration safety.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moeffel/claude-workflow](https://github.com/moeffel/claude-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
