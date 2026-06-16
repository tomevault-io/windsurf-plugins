---
trigger: always_on
description: This document is the agent contract for this repo. It applies identically to Codex, Claude Code, and any other agentic CLI working here. `CLAUDE.md` is a symlink to this file — do not edit them independently.
---

# AGENTS

This document is the agent contract for this repo. It applies identically to Codex, Claude Code, and any other agentic CLI working here. `CLAUDE.md` is a symlink to this file — do not edit them independently.

## Objective

- Optimize for task completion with low token use.
- Prefer phase-based execution over conversational micro-steps.

## Claude Code quickstart

If you are a Claude Code session arriving in this repo for the first time:

1. **Branch awareness** — by default ANY branch that is not a protected base
   (`main`/`dev`/`master`, plus any repo-configured protected branch) counts as
   an agent-managed branch you may edit and commit on. `agent/*`, `claude/*`,
   `vendor/*`, `feat/*`, or any ad-hoc name all work — being OFF a protected
   base is the only load-bearing rule, so you don't need to set
   `GUARDEX_AGENT_BRANCH_PREFIXES`. Lockdown is opt-in: set
   `GUARDEX_AGENT_BRANCH_PREFIXES_ONLY=1` (+ an explicit prefix list) to gate
   the Claude Code edit/Bash guard, and/or `GUARDEX_REQUIRE_AGENT_BRANCH=1`
   (or `git config multiagent.requireAgentBranch true`) to force git commits
   back onto the `agent/*` namespace.
2. **Slash commands** — `/gx-status`, `/gx-doctor`, `/gx-pivot`,
   `/gx-pr`, `/gx-finish`, `/gx-setup`, `/gx-act` are available out of the
   box. See `.claude/commands/`. `/gx-act` wraps
   [nektos/act](https://github.com/nektos/act) so CI workflows run locally
   before the remote PR run, letting you squash-merge on the first green
   round-trip.
3. **PR flow** — when you need explicit PR control, use `gx pr open`,
   `gx pr status`, `gx pr sync`, or `gx pr watch`. For end-of-task
   commit + push + PR + merge + cleanup, still use the non-negotiable
   `gx branch finish --via-pr --wait-for-merge --cleanup`.
4. **Repo wiring** — `gx claude install` writes `.claude/settings.json`,
   hooks, slash commands, the gitguardex skill, and a `.mcp.json` that registers
   the read-only `gx` MCP server (the cross-repo agent radar: `list_agents`,
   `who_owns`, `my_context`) into a target repo. Opt out with `--no-mcp`.
   `gx claude check` diagnoses drift without writing; `gx claude doctor`
   diagnoses and repairs.

## ExecPlans

When writing complex features or significant refactors, use an ExecPlan (as described in `.agent/PLANS.md`) from design to implementation.

## Quick rules (non-negotiables)

- Never edit, stage, or commit on `dev` / `main`. Open an `agent/*` branch + worktree first.
- Claim files before edits: `gx locks claim --branch "<agent-branch>" <file...>` (or Colony `task_claim_file` on an active task).
- Finish completed work with `gx branch finish --branch "<agent-branch>" --via-pr --wait-for-merge --cleanup`. Never stop at bare `--via-pr`.
- Commit, push, and open/update a PR for completed work unless the user explicitly says to keep it local.
- Use OpenSpec for change-driven work; create/update `openspec/changes/<slug>/` before editing code (helper agent sub-branches excepted).
- Keep outputs compact: less word, same proof.
- Do not commit ephemeral runtime artifacts or local settings: `.dev-ports.json`, `apps/logs/*.log`, `.codex/settings.local.json`, `.claude/settings.local.json`, `.omc/project-memory.json`, `.omc/state/**`, `.omx/state/**`.
- Do not embed stale memory dumps, PR transcripts, session history, or long logs in this file.
- Frontend/UI/UX requests: load `.codex/skills/ui-ux-pro-max/SKILL.md` first.
- The `multiagent-safety` marker section below is machine-managed. Do not edit between markers.

## Workflow cheatsheet

```bash
# 1. Start a sandbox worktree (tier sizes OpenSpec scaffolding):
ALLOW_BASH_ON_NON_AGENT_BRANCH=1 \
  gx branch start [--tier T0|T1|T2|T3] "<task>" "claude-<name>"

# 2. Work inside the printed worktree path:
cd .omc/agent-worktrees/gitguardex__claude-<name>__<slug>
gx locks claim --branch "agent/claude-<name>/<slug>" <file...>
# implement + commit inside this worktree

# 3. Validate specs (before archive / finish on T2/T3):
openspec validate --specs

# 4. Finish via PR + cleanup (the non-negotiable default):
gx branch finish \
  --branch "agent/claude-<name>/<slug>" \
  --base main --via-pr --wait-for-merge --cleanup

# Branch protection blocks merge? Enable auto-merge once PR URL is known:
gh pr merge <PR-NUMBER> --repo <owner>/<repo> --auto --squash

# Sweep multiple finished lanes in one shot:
gx finish --all
```

Tier guide (sized by blast radius; **default is `T1`** when `--tier` is omitted —
escalate with `--tier T2` for a behavior change or `--tier T3` for plan-driven work):

| Tier | Use for | Scaffolding | Gates |
|------|---------|-------------|-------|
| `T0` | typos, dep bumps, format-only | none | tasks gate skipped |
| `T1` | ≤5 files, 1 capability, no API/schema | notes.md only | tasks gate skipped |
| `T2` | behavior change, API/schema, multi-module | full change workspace | full gates |
| `T3` | cross-cutting, multi-agent, plan-driven | change + plan workspace | full gates |

See [`.agent/CLAUDE-CODE-WORKFLOW.md`](.agent/CLAUDE-CODE-WORKFLOW.md) for full tier examples, finish flow, and `skill_guard` notes.

## Environment


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opencue/gitguardex](https://github.com/opencue/gitguardex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
