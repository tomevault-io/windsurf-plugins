---
trigger: always_on
description: This file defines repo-level operating rules for any agent working in Maakdown.
---

# AGENTS

This file defines repo-level operating rules for any agent working in Maakdown.

## Core Rules

1. Follow the approved docs in `docs/`; if implementation must deviate, update `DEV_CONTEXT.md` and the relevant docs.
2. Keep `docs/task-tracker.md` current when tasks are added, reprioritized, started, completed, or blocked.
3. Keep `DEV_CONTEXT.md` current whenever project state meaningfully changes.
4. Do not push to a remote repository, create remote branches, or open pull requests unless the user explicitly asks.
5. Never commit signing certificates, private keys, notarization credentials, store credentials, provisioning profiles, or generated release binaries.

## Verification And Commit Policy

- Every meaningful implementation change should end with verification that matches the scope of the change.
- Check `git status` before editing. The worktree may contain user changes.
- Prefer narrow changes and avoid bundling unrelated cleanup into product work.
- Use the cheapest useful verification first:
  - docs/tree-only changes: `find`, `rg`, and `git status`
  - frontend changes: `npm install` if needed, then `npm run check` and `npm run build`
  - backend changes: `go test ./...` once Go is installed
  - full app changes: `wails build` once Wails and Go are installed
- After verification succeeds, commit the completed coherent unit of work unless the user asks not to.
- Do not batch unrelated work into one commit when they can be split cleanly.
- If verification is blocked by missing local tools or functional constraints, record the blocker in `DEV_CONTEXT.md`.

## GitHub Actions Verification

- Verify changes to GitHub Actions workflows or scripts used directly by those
  workflows on the remote `ci/sandbox` branch before pushing them to `main`.
- Keep `workflow_dispatch` available on the workflow under test.
- Use this flow without creating a local sandbox branch:
  1. Force-push local `HEAD` to `origin/ci/sandbox`.
  2. Trigger the workflow with
     `gh workflow run <workflow-file> --ref ci/sandbox`.
  3. Watch the run for up to 10 minutes and inspect failures before touching
     `main`.
- Retry at most three remote attempts while correcting workflow definitions,
  workflow scripts, runner setup, permissions, artifact handling, or other CI
  infrastructure owned by the change.
- If the run is still active after 10 minutes, stop watching and ask the user
  to report its final result before continuing.
- If the first sandbox attempt succeeds, the verified change may be pushed to
  `main` after local verification.
- If verification requires multiple sandbox attempts, pause for user
  confirmation before pushing to `main`.
- Cross-platform workflow verification establishes that the workflow,
  supporting scripts, tests, and builds execute on the declared runners. Do
  not fix unrelated product behavior merely to turn a workflow run green;
  report product failures separately unless the user expands the task.

## DEV_CONTEXT.md Policy

`DEV_CONTEXT.md` is required project memory. Update it whenever one or more of the following changes:

- the purpose of files or directories becomes clearer
- a new file or subsystem is added
- a planned task is added, removed, reprioritized, or redefined
- a task is completed
- an implementation detail materially changes
- an architectural or product decision is made or revised
- verification commands or blockers change

At minimum, `DEV_CONTEXT.md` should contain:

- project summary
- tree-level summary
- current phase and active focus
- planned tasks and features
- completed tasks with timestamps
- implementation notes and decision log
- current verification commands and blockers

## Task Tracker Policy

`docs/task-tracker.md` is the project/progress tracker. It should be updated as work lands:

- `Todo`: not started
- `In Progress`: actively being worked
- `Blocked`: cannot proceed without a dependency or decision
- `Done`: implemented and verified
- `Deferred`: intentionally out of current scope

Each task should have an owner field, dependencies, exit criteria, and verification notes.

## Remote Policy

- Never push.
- Never create or update a remote branch.
- Never open a pull request.
- Exception: only do any of the above if the user explicitly instructs you to.

## Signing And Release Policy

The user plans to sign macOS and Windows builds with their own certificates. Agents must preserve that structure without storing secrets:

- macOS signing assets belong in the user's keychain or external secret storage, not git.
- Windows code-signing certificates (`.pfx`, `.p12`, `.cer`, private keys) must not be committed.
- Notarization credentials, Apple API keys, timestamp server credentials, and CI secrets must be referenced by environment variable names or secret-manager keys only.
- Keep signing-safe templates, entitlements, manifests, and documentation in git.
- Generated installers, disk images, signed binaries, symbol archives, and notarization logs are build artifacts and should remain ignored unless the user explicitly asks otherwise.

## Release Guide Policy

- When asked to prepare, cut, sign, publish, verify, document, or complete a
  release, read and follow `docs/RELEASING.md` as the canonical release
  runbook.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cybermaak/maakdown](https://github.com/cybermaak/maakdown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
