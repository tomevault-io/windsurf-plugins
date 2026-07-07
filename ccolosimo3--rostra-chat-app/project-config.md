---
trigger: always_on
description: `AGENTS.md` is the canonical entrypoint for agents and contributors in this repository.
---

# AGENTS.md — Rostra

## Start Here (Canonical Entrypoint)

`AGENTS.md` is the canonical entrypoint for agents and contributors in this repository.

Read in this order:
1. `AGENTS.md` (this file)
2. `WORKFLOW.md`
3. `docs/ISSUES_WORKFLOW.md`
4. `docs/ARCHITECTURE.md`
5. `docs/PATTERNS.md`
6. `docs/REVIEW_CHECKLIST.md`
7. `skills/write-spec.md`
8. `skills/spec-to-issues.md`
9. `skills/issue-to-pr.md`
10. `skills/spec-workflow-gh.md`

## Unit of Work Rule

- **Unit of work is a GitHub Issue.**
- Choose an execution mode from `docs/ISSUES_WORKFLOW.md` before coding:
  - `single` (default): one feature -> one Task issue -> one PR
  - `gated`: Spec issue + child Task issue(s) for feature sets or higher-risk work
  - `fast`: quick-fix path for tiny low-risk changes
- Convert freeform requests into the selected issue mode before implementation.
- Work one Task issue at a time.
- PRs close Task issues (`Closes #123`), not Specs.
- Specs close only when all child Tasks are done or explicitly deferred.
- Detailed control-plane rules are canonical in `docs/ISSUES_WORKFLOW.md`.
- For one-shot issue body + `gh` command generation, use `skills/spec-workflow-gh.md`.
- Canonical single-line kickoff prompt:
  - `Run kickoff for feature <feature-id> from <filename> mode=<single|gated|fast>.`
  - If `mode` is omitted, default to `single`.
  - Expected output: issue body file(s), `gh issue create` command(s), created issue link(s), and a 3-5 step implementation plan.

## Agent Operating Loop

1. Whiteboard scope in `plans/*.md` or a spec doc section (scratch only).
2. Choose execution mode (`single` default, `gated`, or `fast`) and create required issue(s).
3. Restate goal and acceptance criteria.
4. Plan minimal files and scope.
5. Implement with tight, surgical changes.
6. Run verification commands.
7. Update tests/docs if required.
8. Open PR that closes the Task issue.
9. Run one fresh-context review pass and patch notable findings on the same branch (bounded by workflow caps).
10. Close Spec after child Tasks are done/deferred.

## Process

Read and follow `WORKFLOW.md` for the full development process and `docs/ISSUES_WORKFLOW.md` for the issue-control execution modes. Together they define the Design → Test → Implement → Review → Document loop, TDD workflow, technical constraints (SQLAlchemy 2.0, Pydantic v2, async patterns), security requirements, and documentation maintenance rules.

This file contains **project-specific rules** that supplement WORKFLOW.md. If they conflict, this file wins.

---

## Project Context

Rostra is a real-time chat application with multi-room support, unread message tracking, and WebSocket-based messaging.

**Stack:** FastAPI (Python) backend, React + TypeScript + Vite SPA frontend, PostgreSQL, Redis (caching/rate limiting), WebSockets for real-time messaging.

**This is a Vite SPA with react-router-dom, NOT Next.js.** No server components, no `'use client'` directive, no App Router. WORKFLOW.md uses Next.js in its examples — adapt those patterns to Vite/React-Router for this project.

**Deployment:** Vercel (frontend), Render (backend), Supabase (database).

---

## Core Rules

These behavioral rules apply to every task in this project, in addition to WORKFLOW.md's agent operating rules.

- **Simplicity first.** Write the minimum code that solves the problem. No features beyond what was asked. No abstractions for single-use code. No speculative "flexibility" or "configurability." No error handling for impossible scenarios. If you write 200 lines and it could be 50, rewrite it. Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.
- **Surgical changes only.** Touch only what the task requires. Don't "improve" adjacent code, comments, or formatting. Don't refactor things that aren't broken. Match existing style, even if you'd do it differently. If you notice unrelated dead code, mention it — don't delete it. Clean up only orphans YOUR changes created (unused imports, variables, functions). Every changed line should trace directly to the user's request.
- **Explain what you're doing.** When writing or modifying code, include brief comments explaining *why* (not just what) for any non-obvious logic. This is a learning environment.
- **Prefer explicit over clever.** Write readable, straightforward code. No one-liners that sacrifice clarity. No premature optimization. No unnecessary abstractions.

## Decision Brief (Required)

For non-trivial fixes/features, include a short decision brief before completion:

- **Chosen approach:** what was implemented.
- **Alternative considered:** one realistic alternative.
- **Tradeoff:** why this choice won (complexity/risk/perf/security).
- **Revisit trigger:** when the alternative should be reconsidered.

For tiny quick fixes, a one-line brief is enough: chosen approach + primary risk.

---

## Verification

Before considering any task complete, run the relevant checks:

### Backend (FastAPI/Python)
Preferred: run from repo root:
```bash
make backend-verify
```

If the environment cannot reach PostgreSQL (e.g., restricted sandbox), run:
```bash
make backend-verify SKIP_DB_BOOTSTRAP=1
```
This skips DB-backed test fixtures; DB-dependent tests will be skipped.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ccolosimo3/rostra-chat-app](https://github.com/ccolosimo3/rostra-chat-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
