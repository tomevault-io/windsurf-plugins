---
trigger: always_on
description: >-
---


# Swarm Review

A coordinated swarm of specialized AI code reviewers. Spawns domain-specific sub-reviewers in parallel, then a coordinator deduplicates, re-categorizes, and judges the final verdict.

---

## Quick Install by Harness

| Harness | Install | Invoke |
|---------|---------|--------|
| **npm CLI** | `npm install -g swarm-review` | `swarm-review HEAD~1 --model deepseek-v4-flash --provider deepseek` |
| **Copilot CLI** | Copy skill dir to `~/.agents/skills/swarm-review/` | *"swarm review"* or *"review this PR"* |
| **pi coding agent** | Copy this file to `~/.pi/agent/skills/swarm-review.md` | `/skill:swarm-review` or *"swarm review"* |
| **Claude Code** | Copy this file to `AGENTS.md` in your repo root | *"run a swarm review"* |
| **opencode** | Copy this file to `AGENTS.md` in your repo root | *"swarm review"* |

> All reviewer prompts are embedded inline in the **Reviewer Prompts** section below.

---

## Orchestration

> **YOU ARE THE ORCHESTRATOR — NOT THE REVIEWER.**
> Your job is to set up context, spawn sub-agents, collect their output, and present the verdict.
> You MUST NOT read source files, analyse code, or produce findings yourself.
> Every finding MUST come from a sub-agent spawned via the `subagent` tool.
> Skipping sub-agent spawning and doing the review inline defeats the entire purpose of this skill.

### Phase 0 — Detect & Confirm Review Target

Run these commands to determine what to review:

```bash
git rev-parse --show-toplevel   # repo root
git branch --show-current       # current branch
git status --porcelain          # uncommitted changes?
git log --oneline -5            # recent context
```

Select the review target using this priority table:

| Condition | Review Target | Command |
|-----------|--------------|---------|
| Uncommitted changes exist | Working tree diff | `git diff HEAD > .swarm-review/diff.patch` |
| On feature branch, clean | All commits since `main`/`master` | `git diff $(git merge-base HEAD main) HEAD > .swarm-review/diff.patch` |
| On `main`/`master`, clean | Last commit | `git diff HEAD~1 HEAD > .swarm-review/diff.patch` |

**Before confirming, parse the user's invocation phrase for reviewer constraints.**
If the user mentioned specific domains — e.g. *"code quality and security"*, *"just performance"*, *"security only"* — record those as the active reviewer list. They override the tier roster entirely; spawn only those reviewers.

**Ask the user to confirm** the detected target. Show which reviewers will run (either from the invocation or from the tier). Invite additional custom instructions (focus area, skip list, hotfix flag). Custom instructions are appended to every reviewer prompt.

---

### Phase 1 — Assess Risk Tier

Parse the diff and classify:

```
totalLines  = sum of added + removed lines across all files
fileCount   = number of changed files
securityHit = any file path contains: auth/, crypto/, oauth, jwt, session,
              password, credential, token, secret, ssl, tls, encrypt, decrypt,
              permission, rbac, acl, authentication, authorization
```

| Tier | Condition | Reviewers to spawn |
|------|-----------|-------------------|
| **trivial** | ≤10 lines AND ≤20 files | coordinator + code-quality |
| **lite** | ≤100 lines AND ≤20 files | + documentation + agents-md |
| **full** | >100 lines OR >20 files OR securityHit | all 7 |

Security-sensitive files always trigger **full** regardless of diff size.

---

### Phase 2 — Filter Noise

Strip the following from the diff before reviewers see it:

- Lock files: `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `bun.lock`, `Cargo.lock`, `go.sum`, `poetry.lock`, `Pipfile.lock`, `flake.lock`
- Minified/bundled assets: `*.min.js`, `*.min.css`, `*.bundle.js`, `*.map`
- Files whose first 5 lines contain `@generated` or `// eslint-disable` (except database migrations)

---

### Phase 3 — Spawn Sub-Reviewers (Parallel)

**Call `subagent({ tasks: [...] })` now.** Do not read source files yourself. Do not produce findings inline.
Every reviewer runs as a separate sub-agent with `agent: 'worker'`. All tasks in the array run concurrently.

**Reviewer selection — in priority order:**
1. If the user named specific domains in their invocation or during Phase 0 confirmation, spawn ONLY those reviewers — ignore the tier roster entirely.
2. Otherwise, use the tier roster below.

Each task receives:
1. The full reviewer prompt — **inlined directly into the `task` string by the orchestrator**. Do not reference SKILL.md in `reads`. Copy the relevant reviewer section from this file into the task string.
2. The diff and shared context as `reads` — workspace-relative paths only.

Each sub-reviewer writes plain-text findings to `.swarm-review/reports/<name>-findings.md`.

**Roster by tier — spawn exactly these tasks:**

| Reviewer | trivial | lite | full |
|----------|---------|------|------|
| code-quality | ✓ | ✓ | ✓ |
| documentation | | ✓ | ✓ |
| agents-md | | ✓ | ✓ |
| security | | | ✓ |
| performance | | | ✓ |
| codex | | | ✓ |
| release | | | ✓ (only when release files touched) |

**Example `subagent` call — prompt inlined, no SKILL.md in reads:**

```js
await subagent({
  tasks: [
    {
      agent: 'worker',
      task: `<full text of the Security Reviewer section from this file>

Diff to review: .swarm-review/diff.patch

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kahdeg-15520487/swarm-review](https://github.com/Kahdeg-15520487/swarm-review) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
