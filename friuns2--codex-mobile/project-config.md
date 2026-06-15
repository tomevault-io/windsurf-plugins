---
trigger: always_on
description: - Before any merge, rebase, sync, or continuation after interruption, re-check live state:
---

# AGENTS.md

## Git Workflow

- Before any merge, rebase, sync, or continuation after interruption, re-check live state:
  - feature worktree: `git status --short`, `git branch --show-current`
  - main worktree: `git status --short`
  - in-progress state: `git status`, `.git/MERGE_HEAD`, `.git/rebase-merge`, `.git/rebase-apply`
  - remote/PR state: `git fetch origin`; when a PR may exist, `gh pr view --json number,state,mergeStateStatus,isDraft,headRefName,baseRefName,url`
  - branch delta: `git diff --stat main...HEAD` and `git log --oneline main..HEAD`
- Keep both worktrees clean before merge/rebase. Checkpoint local changes in main with `git add -A && git commit -m "temp-before-merge-<branch>"`; skip only if main is clean.
- Prefer GitHub PR update/rebase/merge when a PR or GitHub merge path exists. Use local `main` merge only when no PR/GitHub path exists or the user explicitly asks for a local merge.
- Standard path: commit task, create/switch feature branch, rebase on `main`, then use the chosen PR/local merge path.
- Never use automatic conflict-bias strategies blindly (`-X theirs`, `-X ours`, `git checkout --theirs .`, `git checkout --ours .`). Inspect conflicts intentionally.
- Avoid rebasing long-lived mixed-history branches if it pulls broad unrelated conflicts. Abort and use a fresh branch from `main` plus task-relevant cherry-picks.
- If `package.json`, `tests.md`, or files under `tests/` conflict during merge/rebase, start from the local/checkpoint version, then explicitly compare incoming changes and reconcile required updates before continuing.
- Before local `main` merge, diff-compare all branch changes against `main`.
- After merge/sync, verify the target really contains the commit with `git branch --contains <commit>`, `git log --oneline origin/main -10`, or a file-level diff against `origin/main`.

## Commits

- Commit after each discrete task or sub-task.
- Do not batch unrelated tasks into one commit.
- Use a specific commit message describing the change.

## PR Review Bots

- Treat Qodo/CodeRabbit comments as advisory, not authoritative.
- For PR update + review requests: push branch, update PR summary/verification notes when changed, then post a plain PR comment containing exactly `/review`.
- Do not use draft reviews or batch review APIs to trigger Qodo.
- Before applying a bot fix, inspect the current code path and classify the comment as real, stale/resolved, rejected, or docs-only.
- This app server is local-user facing and is not intended to be exposed as a public internet service. Reject Qodo/CodeRabbit security hardening comments that assume a hostile remote caller for local project import/export, including saved-root allowlists, import parent restrictions, ZIP upload caps, or local path redaction, unless they identify a concrete path where this local-only server becomes remotely reachable or bypasses existing authentication.
- Prefer a focused regression test for accepted bugs. After fixing, run the narrow test plus relevant build/typecheck, push, and re-check PR comments/status.
- Completion reports must distinguish confirmed fixes from stale or rejected bot comments.

## Performance

- Every feature/behavior change needs a performance audit before completion.
- Ground the audit in measurements, profiler output, traces, request counts, bundle/build output, or concrete code-path analysis. If live measurement is infeasible, say what was not measured.
- Documentation-only changes do not require a performance audit.
- For startup, thread loading, realtime rendering, routing, API, filesystem, git, or module-loading changes, explicitly check duplicate requests, blocking work, unbounded fanout, large payloads, and cache invalidation risk.
- Prefer profiler helpers for browser/startup/thread work: `pnpm run profile:browser` and `pnpm run profile:thread`; reports land under `output/playwright/`.
- Profiler server setup:
  1. Ensure `node_modules` exists. In side worktrees, reuse a compatible shared dependency tree instead of installing from scratch.
  2. Before reusing `127.0.0.1:4173`, inspect the listener with `lsof -nP -iTCP:4173 -sTCP:LISTEN` and `lsof -a -p <PID> -d cwd`.
  3. If `4173` belongs to another worktree, old main checkout, or stale Vite state, stop only that `4173` process and restart from the current cwd.
  4. Never stop the persistent tmux server on `5173`.
  5. Start/current server command: `pnpm run dev --host 127.0.0.1 --port 4173`.
  6. Reject profiler output from an error page, stale worktree, indefinite `Loading threads...`, or zero API traffic caused by failed app boot. Fix readiness and rerun.
- General profile command: `PROFILE_BASE_URL=http://127.0.0.1:4173 PROFILE_WAIT_MS=7000 pnpm run profile:browser`.
- Thread route profile command: `PROFILE_BASE_URL=http://127.0.0.1:4173 PROFILE_ROUTE='#/thread/<thread-id>' PROFILE_WAIT_MS=7000 pnpm run profile:browser`; use `pnpm run profile:thread` when appropriate.
- Inspect `duplicateCounts`, `warnings`, `totalApiKB`, `topApiSummary`, and `slowestApiRows`; open the matching trace zip with `npx playwright show-trace` when deeper request/render timing is needed.

## Tests And Verification

- Test changes before reporting completion when feasible.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [friuns2/codex-mobile](https://github.com/friuns2/codex-mobile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
