---
trigger: always_on
description: This is the `allagents` CLI: a tool for managing AI coding assistant plugins, workspaces, and client synchronization across Claude Code, GitHub Copilot, Cursor, and related clients.
---

# AllAgents Repository Guidelines

This is the `allagents` CLI: a tool for managing AI coding assistant plugins, workspaces, and client synchronization across Claude Code, GitHub Copilot, Cursor, and related clients.

## High-Level Goals

AllAgents should stay predictable, scriptable, and safe for users who manage real local workspaces:
- Keep workspace and plugin operations deterministic.
- Prefer straightforward filesystem transforms over hidden state.
- Make sync/install/update behavior easy to reason about from CLI output alone.
- Preserve user-owned config where ownership boundaries matter.

## Working Style

### Worktree Setup
- For any feature, bug fix, or non-trivial repo change, work from a dedicated git worktree based on the latest `origin/main`.
- Before starting implementation, run `git fetch origin` and verify your worktree `HEAD` is based on the current `origin/main` commit.
- Do not implement from the primary checkout, from a stale local `main`, or from a branch created off an outdated base.
- Default setup:
```bash
git fetch origin
git worktree add ../allagents.worktrees/<type>-<short-desc> -b <type>/<issue-or-topic>-<short-desc> origin/main
cd ../allagents.worktrees/<type>-<short-desc>
bun install
```
- If you discover you are not on a fresh worktree from the latest `origin/main`, stop and fix that first before changing code.

### Planning
- Use plan mode for any non-trivial task (5+ steps or architectural decisions).
- If something goes sideways, STOP and re-plan immediately instead of pushing through a broken approach.
- For non-trivial changes, pause and ask: "Is there a more elegant solution?" before diving in.
- Check in with the user before starting implementation on ambiguous tasks.
- Prefer automation: execute the requested work without extra confirmation unless blocked by missing information, safety concerns, or an irreversible/destructive action the user has not approved.

### Bug Fixes
- Before writing code for a bug fix, confirm you understand the actual problem.
- Ask clarifying questions when the issue is vague, not reproducible from the description, could be user error, or suggests a solution that may not match the real failure.
- Do not assume the first plausible code path is the root cause. Verify the exact commands, environment, and expected versus actual behavior first.

### Subagent Strategy
- Use subagents aggressively to keep the main context window clean.
- Subagents are useful for research, file exploration, tests, and code review.
- For complex problems, parallelize independent investigation and validation work where possible.
- Run a final code review after implementation is complete and before the final green E2E when the task is substantial.

### Simplicity
- Every change should be as simple as possible. Reuse existing code before introducing new abstractions.
- Fix root causes directly. Avoid shotgun debugging and speculative infrastructure.
- Prefer changes that keep ownership boundaries obvious, especially around synced config and generated artifacts.

### Progress Updates
- Provide high-level status updates at natural milestones.
- When scope changes mid-task, communicate the shift and adjust the plan.
- Use parallel tool calls when applicable, especially for independent reads, checks, and validation steps.

### Plans
- Temporary plans and design notes may live under `.claude/plans/` while work is in progress.
- Once implementation is complete, delete stale plan files and update durable docs for any user-facing behavior changes.

## PR & Commit Titles
- Prefer conventional commit style for branch-facing titles: `type(scope): summary`.
- Use the repository's normal types where they fit, such as `feat`, `fix`, `docs`, `style`, `refactor`, `test`, and `chore`.
- Use the most relevant area as `scope`, such as `cli`, `sync`, `plugin`, `workspace`, `docs`, or `mcp`.
- Do not prefix PR titles with `[codex]` unless the user explicitly requests it.
- Do not add `Co-Authored-By` attribution unless explicitly requested.

## Branch & PR Workflow

### Branching
- Never implement directly on `main` for feature or bug-fix work. Use a branch and PR.
- When working from an issue, create the branch from a fresh worktree:
```bash
git worktree add ../allagents.worktrees/<type>-<short-description> -b <type>/<issue-number>-<short-description> origin/main
cd ../allagents.worktrees/<type>-<short-description>
```

### Pull Requests
- Push the branch and open a PR once the change is validated.
- When referencing an issue, include `Closes #<issue-number>` in the PR body.
- Before merging, ensure CI passes, the branch is up to date enough to merge cleanly, and any required review has happened.
- Include the exact E2E steps in the PR description so a reviewer can reproduce what was validated.

### Merge Policy
- Always use squash merge when merging PRs to `main`.
- Do not use regular merge or rebase merge unless the user explicitly asks for it.
```bash
gh pr merge <PR_NUMBER> --squash --delete-branch --admin
```

### After Squash Merge
- After a PR is squash-merged, do not continue working from the old branch.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EntityProcess/allagents](https://github.com/EntityProcess/allagents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
