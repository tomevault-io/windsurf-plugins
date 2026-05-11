---
trigger: always_on
description: This file is authoritative for the MCP server, plugin, hub, and repo-level tooling. For web app conventions (Next.js 15 app under `apps/web/`), see `apps/web/AGENTS.md`.
---


## Scope

This file is authoritative for the MCP server, plugin, hub, and repo-level tooling. For web app conventions (Next.js 15 app under `apps/web/`), see `apps/web/AGENTS.md`.

## Development Workflow (Source of Truth)

Every unit of work runs through a mandatory skill. Two additional protocols activate conditionally.

### Mandatory Skills

**`hdd`** — Hypothesis-Driven Development for architectural decisions, new features, protocol implementations, and behavior-changing refactors. Produces a staging ADR + spec pair that must be validated before accepting. Code is an implementation artifact; ADRs are the source of truth.

Read `.claude/skills/hdd/SKILL.md` when the work requires an architectural decision.

### Conditional Protocols

**`ulysses-protocol`** — Activates when 2 consecutive surprises occur during a task. A surprise-gated debugging framework that prevents hallucinated progress through pre-committed recovery actions and falsifiable hypotheses.

Invoke only when stuck: `.claude/skills/ulysses-protocol/SKILL.md`

**`theseus-protocol`** — Use when the task is a refactor (structure changes, behavior preserved). Prevents Refactoring Fugue State via scope locking, adversarial Cassandra audits, and hard reversibility. Do not use for feature work or bug fixes.

Invoke only for refactoring tasks: `.claude/skills/theseus-protocol/SKILL.md`

### Key Rules (always apply)

1. **Specs go in `.specs/`** (not `specs/`). ADRs use the HDD lifecycle: `.adr/staging/` → `.adr/accepted/` or `.adr/rejected/`.
2. **Code and spec updates in the same commit.** If you change code that a spec describes, update the spec in the same commit.
3. **Atomic commits.** One sub-agent = one unit of work = one commit, made after review validates the work.
4. **Sub-agent summaries state**: Claims, Hypothesis Alignment, Tests run, Known Gaps, Risks.
5. **Default: human is NOT in the loop.** Operate autonomously. Escalate only when genuinely stuck after investigation.
6. **Orchestrators don't do manual work.** Deploy sub-agents or agent teams. Protect your context window.

### References

- HDD process: `.claude/skills/hdd/SKILL.md`
- Ulysses protocol: `.claude/skills/ulysses-protocol/SKILL.md`
- Theseus protocol: `.claude/skills/theseus-protocol/SKILL.md`

## Branch Rules for Agents

This project uses **GitHub Flow**: short-lived feature branches off `main`, one PR per unit of work, merge when green. No long-lived integration branches. See `docs/WORKFLOW-MASTER-DESCRIPTION.md` § Branching Strategy for full rationale.

Agent-specific enforcement rules:

1. **Before first commit: verify branch scope matches work.**
   - `git branch --show-current` — check where you are
   - `fix/X` branches are for fixing X — not for new features
   - `feat/X` branches are for feature X — not for unrelated fixes
   - If scope doesn't match, create a new branch from `main`
2. **After PR is merged: delete the branch** (local + remote). This is not optional.
3. **Never create branches with timestamps, UUIDs, or auto-generated suffixes.**
4. **Never commit to `main` directly.**
5. **Plans must include branch creation as Step 0** when the work is a new unit.

Committing unrelated work to an existing branch pollutes PRs, makes reverts dangerous, creates merge conflicts, and makes git history useless for archaeology. **This is non-negotiable.**

## Landing the Plane (Session Completion)

**When ending a work session**, you MUST complete ALL steps below. Work is NOT complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1. **File issues for ALL remaining work** - Every follow-up, deferred decision, or "next session" item MUST be tracked before the session ends. If an ADR references future work (e.g., "deferred to ADR-010"), create the tracking item now.
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Update issue status** - Close finished work, update in-progress items
4. **PUSH TO REMOTE** - This is MANDATORY:
   ```bash
   git pull --rebase
   git push
   git status  # MUST show "up to date with origin"
   ```
5. **Clean up** - Clear stashes, prune remote branches
6. **Verify** - All changes committed AND pushed
7. **Hand off** - Provide context for next session

**CRITICAL RULES:**
- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing - that leaves work stranded locally
- NEVER say "ready to push when you are" - YOU must push
- If push fails, resolve and retry until it succeeds


## Local Agent Asset Bridge (`.codex/`, `.claude/`, and `.gemini/`)

These directories contain project-local agent instructions. Codex may not
automatically discover project-local `.codex/` skills unless the user config
loads this repo path as a skill root. Claude/Gemini hooks and slash commands
also cannot be natively installed by Codex. Treat these assets as **manual
operating instructions** for this repo when they match the task.

### Resolution Order

When these sources disagree, use this order:

1. `AGENTS.md`
2. `.codex/skills/`
3. `.claude/skills/` and `.claude/commands/`
4. `.gemini/skills/` and `.gemini/commands/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kastalien-Research/thoughtbox](https://github.com/Kastalien-Research/thoughtbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
