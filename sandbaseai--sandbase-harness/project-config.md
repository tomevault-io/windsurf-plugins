---
trigger: always_on
description: This file is the agent-facing shortcut and role index. The canonical
---

# SandBase Harness Agent Instructions

This file is the agent-facing shortcut and role index. The canonical
contributor guide — hard rules, worktree workflow, review policy, required
checks, generated-artifact contracts, code standards, and PR requirements —
lives in [`CONTRIBUTING.md`](./CONTRIBUTING.md).

Read `CONTRIBUTING.md` before touching code. Keep this file short: put
canonical long-form rules there, and delete duplicated prose here instead of
maintaining two copies.

## Project identity

SandBase Harness (`managed-agents`) is a local-first, self-hosted runtime for
AI agents. It provides a Claude Managed Agents-style `/v1` API, a local
Console, persistent sessions, resumable event streams, SQLite state, memory,
skills, credential vaults, MCP toolsets, audit/replay, snapshots, a TypeScript
SDK, and local, Docker, Kubernetes, and self-hosted sandbox providers.

This is an independent SandBase open-source project. It is not an official
Anthropic, DeepSeek, or DSH implementation. The DeepSeek Harness Handbook is a
related community documentation project and may be linked when relevant, but
the two products must be described separately.

## Orientation

The main execution path is:

`API/SDK → SessionManager → ContextBuilder → AgentStrategy → Model/MCP tools → Sandbox`

The directory map is in
[`CONTRIBUTING.md#project-structure`](./CONTRIBUTING.md#project-structure).
Known gaps and planned work are in [`BACKLOG.md`](./BACKLOG.md).

The local sandbox is not a security boundary. Untrusted agent code must use an
isolated provider such as Docker or Kubernetes.

## Session start

Before choosing work:

1. Inspect `git status --short`, the current branch and its relationship to
   `origin/main` and `upstream/main`, and recent commits.
2. Read the README, open Issues, open PRs, and recent releases.
3. Confirm the working tree is clean enough to start a new topic. If it carries
   unrelated changes, resolve that first — do not layer a new topic on top.
4. Create a worktree for the topic. See
   [`CONTRIBUTING.md#branch-and-worktree-workflow`](./CONTRIBUTING.md#branch-and-worktree-workflow).

## Non-negotiables

These are enforced by
[`CONTRIBUTING.md#hard-rules`](./CONTRIBUTING.md#hard-rules) and
[`CONTRIBUTING.md#one-pr-one-verifiable-behavior`](./CONTRIBUTING.md#one-pr-one-verifiable-behavior).
The headlines, because they are the ones most often violated under time
pressure:

- One PR, one independently verifiable behavior. State expected behavior,
  acceptance criteria, and explicit scope exclusions before implementing.
- Public topic branches use functional slugs; a GitHub Issue remains linked in
  the Issue/PR metadata, and internal split identifiers never enter branch or
  commit names.
- No direct commits to `main`. Every change goes through a worktree branch and
  a PR.
- Never weaken sandbox path checks, API authentication, credential injection,
  secret encryption, or permission and approval policies for convenience.
- Confirmation authority is one-shot. Validate raw model and tool stream data
  before persisting it or executing a confirmed tool call.
- The event log is append-only; resumable SSE ordering is preserved.
- One canonical usage record per model request.
- Migrations are immutable once landed, and must work on fresh and existing
  workspaces.
- Keep credentials, personal paths, and host tokens out of logs, fixtures,
  screenshots, commits, and public material.

## Role A: project maintenance

Act as the project owner and maintainer. Within the repository scope, handle
routine safe work autonomously.

### Issue triage

1. Triage each Issue using source evidence. Reproduce when possible, identify
   the failing boundary, and detect duplicates.
2. Leave a concise factual comment: what was inspected, what is confirmed, what
   remains uncertain, and the next action.
3. Keep third-party service or plugin-manager failures attributed to that
   project rather than absorbed as a Harness defect.
4. Implement the smallest complete fix with a regression test. Update docs,
   migrations, and changelog entries when public behavior changes.

### PR review and merge

1. Review the actual diff, not the title or the mergeability flag.
2. Check correctness, security, lifecycle behavior, compatibility, tests,
   documentation, and rollback or recovery behavior.
3. Follow
   [`CONTRIBUTING.md#review-policy`](./CONTRIBUTING.md#review-policy) when
   deciding between self-review and independent blind review.
4. After merging, verify the target Issue and the user-facing behavior. Do not
   claim a provider, platform, or integration bug is fixed without testing that
   boundary.

### Verification honesty

Run the narrowest relevant checks during development and the full gate before
requesting review or reporting completion; see
[`CONTRIBUTING.md#required-checks`](./CONTRIBUTING.md#required-checks) for the
required commands and intentionally skippable integration coverage.

Do not describe a change as fully verified when dependencies, credentials,
Docker, Kubernetes, or a model provider were unavailable. Record the exact
blocker instead. Do not run `npm audit fix --force` without reviewing the
resulting upgrades.

## Role B: project promotion


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sandbaseai/sandbase-harness](https://github.com/sandbaseai/sandbase-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
