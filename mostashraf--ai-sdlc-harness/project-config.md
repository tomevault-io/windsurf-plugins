---
trigger: always_on
description: This project is a Claude Code harness that orchestrates multi-agent development workflows for User Stories / Issues across multiple repos. Supports multiple work item providers (Azure DevOps, Jira, GitLab, GitHub, Zoho, local Markdown) and git providers (Azure DevOps, GitLab, GitHub, gh-cli, glab-cli) via a provider adapter layer. No application code lives here — only agents, skills, hooks, and context.
---

# AI-Driven Development Workflow

This project is a Claude Code harness that orchestrates multi-agent development workflows for User Stories / Issues across multiple repos. Supports multiple work item providers (Azure DevOps, Jira, GitLab, GitHub, Zoho, local Markdown) and git providers (Azure DevOps, GitLab, GitHub, gh-cli, glab-cli) via a provider adapter layer. No application code lives here — only agents, skills, hooks, and context.

## Quick Start

```
/dev-workflow <Work-Item-ID> [project-name] [team-name]
```

Defaults from `provider-config.md`. See the `/dev-workflow` skill for full workflow documentation.

## Workflow Phases

1. **Requirements Ingestion** — Planner pulls story/issue from configured provider, asks clarifying questions. *No feature branch exists yet.*
2. **Planning & Approval** — Planner proposes approaches, human selects one, plan + test outline + tracker generated — **GATE #1**
   - *Pre-flight runs immediately after GATE #1 clears* (`commands/preflight.md`): the orchestrator reads the tracker's `## Repo Status` section (populated by the Planner per `plan-generator/SKILL.md` Step 7), creates the `<team>/feature/<id>-<slug>` branch in **exactly** the repos the plan named (never in every known repo as a "safe default" — the prior behaviour produced orphan branches in unaffected repos), and commits the plan onto the new feature branch in the single-repo workspace-is-git-repo case. Pre-flight has no human gate.
3. **TDD Development Loop** — For each task: Tester writes failing tests → Developer makes them pass → Reviewer reviews combined diff → squash-merge
4. **Human Approval** — Human reviews full implementation — **GATE #2**
5. **Test Hardening** — Tester fills integration/E2E gaps, enforces ≥ 90% coverage, Reviewer reviews
   - *Static security review (P5.5) runs after Phase 5 completes* (`commands/security-review.md`): the orchestrator dispatches per-repo SAST lanes per `language-config.md`, aggregates findings, and stamps `Security review completed <ts>`. When any finding ≥ medium severity surfaces, **GATE #2.5** fires and the human picks `waive` / `fix-now` / `defer`. See `dev-workflow/SKILL.md` Commands table for invocation rules.
6. **PR Creation** — Reviewer does holistic pre-PR review of entire feature branch (all tasks combined, impl + tests, against plan + conventions) → detailed report shown to human → human approves — **GATE #3**
7. **PR Review Response** *(on-demand, after Phase 6)* — Reviewer challenges each PR comment against plan + acceptance criteria, classifies as VALID/INVALID/PARTIAL → findings report shown to human → human selects which comments to address — **GATE #4** → Planner adds new tasks → re-enters Phase 3 loop for those tasks. Repeatable across multiple review rounds.

**Inter-gate: Ad-Hoc Request Handling** *(on-demand, between any pair of gates)* — Human submits a change request at GATE #2 (instead of `APPROVED`), at GATE #3, or mid-phase via `/dev-workflow request <story-id> "<text>"`. Reviewer triages each request against the approved plan, classifies as IN_SCOPE_BUG / IN_SCOPE_AC_MISS / OUT_OF_SCOPE / PLAN_CONFLICT / DUPLICATE / INVALID → findings shown to human → human confirms in-scope items or chooses expand-scope / defer / withdraw for out-of-scope — **GATE #5** → Planner appends rows under `## Ad-hoc Tasks (Batch <N>)` → re-enters Phase 3 loop for those tasks. Repeatable across multiple batches. See `skills/dev-workflow/commands/handle-request.md`.

### Cross-cutting & utility commands

These run outside the linear 1→7 pipeline. Full invocation contracts live in [`skills/dev-workflow/SKILL.md`](skills/dev-workflow/SKILL.md) (Commands table).

- **P8 `reconcile`** — post-merge cleanup: marks the Story-State `Archived`, renames `tracker.md → tracker.archived.md`, stamps `Merge detected` and `Workflow completed`.
- **P9 `metrics`** — aggregates `Plan approved` / `PR created` / per-task durations into `metrics-report.md` + appends to `ai/_metrics-log.csv`. Auto-triggered at T1 (post-PR), T2 (each review cycle), T3 (post-reconcile); also runnable ad-hoc.
- **R `resume`** — workflow-state recovery after a crash; reads `Recovery-State:` and re-enters the interrupted phase. Pair: `/dev-workflow abort <id>` renames `tracker.md → tracker.aborted.md`.
- **`hotfix`** — re-entry on an `Archived` story; operates on a **clone** so the original `Archived` row is preserved.
- **`migrate`** — one-time `ai/plans/` + `ai/tasks/` → `ai/<YYYY-MM-DD>-<work-item-id>/` layout migration. Run once per workspace; the v2.0 startup gate refuses other commands when the legacy layout is detected.
- **`request`** — see Inter-Gate Ad-Hoc Request Handling above.

## Critical Ownership Rules

- The **Orchestrator** is the sole owner of the task tracker. It updates status after every agent verdict. The tracker stays **uncommitted** until Phase 6.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MostAshraf/ai-sdlc-harness](https://github.com/MostAshraf/ai-sdlc-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
