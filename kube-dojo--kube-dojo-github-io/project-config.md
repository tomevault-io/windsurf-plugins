---
trigger: always_on
description: > This file is read by Codex, Jules, and other AI coding agents working on this repository.
---

# AGENTS.md — Rules for AI Coding Agents

> This file is read by Codex, Jules, and other AI coding agents working on this repository.
> For Claude-specific instructions see `CLAUDE.md`. For the agy (Google lane) workflow, see `.claude/rules/agy-workflow.md` (gemini-cli retired, #2125/#2230).

---

## Global Codex Operating Rules

- Start every task at the repository root and run `git status --short --branch` before editing.
- Preserve user and unrelated changes; do not revert, delete, or clean up work outside the task.
- Use multi-agent delegation for non-trivial work that can run in parallel without shared-file contention.
- Keep worker scopes disjoint, and explicitly tell agents not to revert or overwrite others' changes.
- Use `apply_patch` for manual edits.
- Keep changes scoped to the requested files and behavior.
- Run relevant verification before finalizing, or state why verification was not run.
- Final reports must include changed files, verification performed, and final `git status --short --branch`.

---

## Cold start (first call on a fresh session)

**Do not start with `git log` or a grep sweep.** Issue-driven sessions: read the GitHub issue verbatim first, then run:

```bash
KUBEDOJO_ISSUE=N bash scripts/cold-start.sh   # issue reminder + orient in one call
# or without an issue:
bash scripts/cold-start.sh
```

The script brings up services, prints `git status`, pending Decision Cards, then API blocks:
`briefing` (compact snapshot), `orient` (primary action + alternatives), `session` (handoff pointer).
Optional route discovery: `bash scripts/cold-start.sh --manifest`.

Full copy-paste ritual: [`scripts/prompts/cold-start.md`](scripts/prompts/cold-start.md).

Before claiming work: `GET /api/pipeline/leases`. Before fixing a module: `GET /api/module/{key}/state` (structured `diagnostics[]`). Before re-reviewing: `GET /api/reviews?module={key}`. Situational awareness: `GET /api/tracks/readiness` + `GET /api/activity`.

If the API is down, the script **exits 0** with a `STATUS.md` excerpt and handoff path — then read `CLAUDE.md` only if that block is insufficient. More recipes: [`scripts/agent_onboarding.md`](scripts/agent_onboarding.md).

---

## MANDATORY PRE-SUBMIT CHECKLIST

**Before opening a PR, verify EVERY item. If ANY check fails, fix it BEFORE submitting.**

- [ ] `.venv/bin/ruff check` clean on every Python file you changed
- [ ] `.venv/bin/python scripts/test_pipeline.py` — 0 new failures (2 pre-existing `check_failures` tests + 1 `TestStatusFourStage` order flake are acceptable until their dedicated cleanup lands)
- [ ] `npm run build` passes if you touched content under `src/content/docs/` or Astro config (skip for pure-Python-script changes)
- [ ] No `sys.executable` anywhere — always `.venv/bin/python` explicitly
- [ ] No `@pytest.mark.skip` with empty `pass` bodies, no double-skip decorators
- [ ] Assertions not weakened (no `is True` → `isinstance(..., bool)`)
- [ ] Every changed file is directly related to the task
- [ ] Total files changed < 20 (if more, you likely included artifacts)
- [ ] Diff budget respected (tasks specify a LOC ceiling; if you can't fit, split)
- [ ] Primary repo is NOT on detached HEAD after your work

**If you cannot check every box, your PR will be rejected.**

### Book-only AI History PR exception

For PRs that only touch AI History book/research material and do not affect executable code, generated state, or the published Astro site, do **not** run the expensive curriculum pipeline gate. These PRs include changes strictly limited to:

- `docs/research/ai-history/**` (including all narrative drafts, workflow, and coordination docs within this path)

Required checks for these book-only PRs are:

- [ ] `git diff --check` clean on the changed files
- [ ] cross-family review posted as a PR comment
- [ ] no generated artifacts included
- [ ] primary repo remains on `main`

Skip `.venv/bin/python scripts/test_pipeline.py` for this category. That test suite validates the curriculum pipeline and has low signal for unpublished book research while consuming substantial local and model resources. If a book PR also changes Python, scripts, pipeline behavior, `src/content/docs/`, Astro config, or shared tooling, this exception does not apply and the full checklist above is required.

---

## Non-Negotiable Rules

These rules are ABSOLUTE. Violating any one of them results in immediate PR rejection.

### 1. NEVER push or work directly on `main`

Use git worktrees under `.worktrees/<short-name>` on a new branch. Create with:
```bash
git worktree add .worktrees/<name> -b <branch-name> main
```

The primary checkout (`/Users/krisztiankoos/projects/kubedojo`) must always stay on `main`, never detached, never dirty with uncommitted work-in-progress.

### 2. ALWAYS build from the primary main checkout — NOT a worktree

`npm run build` fails inside `.worktrees/*` because Astro's `node_modules` symlink resolution walks `../../node_modules` one level too shallow. When you need to verify the build:
1. Commit your worktree changes
2. Fetch the branch from the primary checkout
3. Run `npm run build` there

This regression bit PR #282. Don't repeat it.

### 3. ALWAYS use `.venv/bin/python`, not `sys.executable` or `python3`

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kube-dojo/kube-dojo.github.io](https://github.com/kube-dojo/kube-dojo.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
