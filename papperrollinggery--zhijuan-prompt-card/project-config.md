---
trigger: always_on
description: - Keep GitHub release tags machine-simple, for example `v0.3.1`.
---

# Project Instructions

## Release Communication

- Keep GitHub release tags machine-simple, for example `v0.3.1`.
- Keep GitHub release titles short because the extension displays `release.name` inside update notices. Prefer titles like `v0.3.1` or `v0.3.1 · API Compatibility`.
- Put detailed update descriptions in `CHANGELOG.md` and the GitHub release body, not in the release title.
- Release notes should compare the key user-visible changes against the previous version without exposing internal prompt wording or low-level implementation details.
- When applicable, mention in-extension update notices and upgrade guidance in both `CHANGELOG.md` and the GitHub release body.
- Thank issue reporters, PR contributors, and model-compatibility testers in public release-facing communication when applicable.

## Gstack + Codex Thread Workflow

- When the user invokes `$gstack`, asks for thread orchestration, or asks for planning/review/QA discipline, run this project through a gstack-style execution plane.
- The current thread is the CEO/control thread. It owns the goal, truth state, role assignment, worker-thread instructions, output merge, final integration, verification, stale-thread cleanup, and final communication.
- The CEO/control thread is not the default implementation executor. For substantial work, delegate implementation to an `Executor` worker in an isolated worktree, or state why a main-thread integration pass is the smaller safe move.
- Use Codex worker threads, not subagents, when the user asks for threads. Each worker gets a role, scope, read/write boundary, expected output format, and verification target.
- Default worker threads are read-only. Write-capable workers require an explicit isolated-worktree merge plan. No worker result is authoritative until the CEO/control thread reads it, reconciles conflicts, and verifies against the final diff.
- Archive worker threads after their report has been merged, or when they are stale, dead, obsolete, failed, or no longer match the accepted scope. Keep the CEO/control thread active.
- Report gstack completion using `DONE`, `DONE_WITH_CONCERNS`, `BLOCKED`, or `NEEDS_CONTEXT`, with evidence.

### Role Matrix

- `Planner / plan-ceo-review`: checks user outcome, premise, scope boundary, alternatives, acceptance criteria, and whether the task is solving the right problem.
- `Engineer / plan-eng-review`: checks architecture, contracts, existing-code leverage, type boundaries, migration risk, and test strategy.
- `Executor / implementation`: applies the accepted plan only. It does not change scope, silently refactor, or self-approve its own work.
- `QA / qa`: verifies behavior with targeted regression first, then broader gates. Use browser evidence when the change is user-visible or URL-driven.
- `Reviewer / review`: performs independent cold review of the final diff for scope drift, regressions, LLM trust-boundary issues, missing tests, and stale docs.
- `Release / ship-documentation`: handles changelog, release notes, update notices, and public wording without exposing internal prompt mechanics.
- `Loop steward`: tracks repeated failures, stale workers, unresolved decisions, and cleanup. The CEO/control thread owns this unless explicitly delegated.

### Gstack Skill Mapping

- Use `/plan-ceo-review` semantics for premise, scope, acceptance criteria, alternatives, and explicit user approval before adding scope.
- Use `/plan-eng-review` semantics for architecture, contracts, affected files, test strategy, rollback risk, and reuse of existing project patterns.
- Use `/review` and `/codex` semantics for independent adversarial review of the final diff, especially LLM trust-boundary and prompt-handoff behavior.
- Use `/qa` semantics for targeted regression first, then broader checks. Browser or UI evidence is required when the change affects visible extension behavior.
- Use `/ship` semantics for branch, PR, GitHub checks, cloud Codex/GitHub review, and merge gating. PR merge always requires explicit user approval.
- If a sub-skill requirement conflicts with this project file, follow the stricter rule and record the reason in the CEO/control thread.

### Adversarial Council Mode

- Run the full council when the user asks for gstack/thread discipline, when behavior crosses the prompt trust boundary, when repeated review feedback appears, when UI-visible handoff behavior changes, before a ship/PR decision after a meaningful diff, or when branch/worktree/merge risk is non-trivial.
- The minimum council roles are `Real user`, `Developer`, `CTO`, `Cold reviewer`, `UI/UX architect`, and `Product manager`. Add `Release` only when public wording, changelog, update notices, or GitHub release content changes.
- Each council worker must be read-only unless explicitly assigned an isolated worktree. Its report must include `STATUS`, blocker findings or PASS rationale, evidence commands or paths, and unresolved risks.
- Do not blindly rerun the full council for tiny mechanical follow-up changes after a clean council pass. For those deltas, run targeted QA plus cold review, and state why the reduced review is sufficient.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [papperrollinggery/zhijuan-prompt-card](https://github.com/papperrollinggery/zhijuan-prompt-card) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
