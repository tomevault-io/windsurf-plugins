---
trigger: always_on
description: ﻿always respond in 中文
---

﻿always respond in 中文

# Harness Hub Instructions

Harness Hub initializes and governs repo-local agent harnesses across multiple projects. Most inner skills may come from upstream sources and should keep their upstream style by default; this repo mainly owns routing, source records, harness templates, lifecycle tooling, and a small number of custom workflow skills.

Keep every distributed skill in the standard layout under `skills/<skill-name>/SKILL.md` with optional `references/`, `scripts/`, and `assets/` spokes.

## Core Rules

1. Think before coding.
   - State assumptions when the request is ambiguous.
   - Surface tradeoffs instead of silently choosing a risky interpretation.
   - Ask only when a reasonable implementation assumption would be unsafe.

2. Simplicity first.
   - Add the minimum code or documentation needed for the requested behavior.
   - Avoid speculative abstractions, options, and host-specific branches.

3. Surgical changes.
   - Touch only files tied to the request.
   - Preserve unrelated worktree changes.
   - Match the existing style of the file you edit.

4. Goal-driven execution.
   - Define success criteria before broad changes.
   - Verify with the nearest tests and validation gates before handoff.

5. PR closeout is part of delivery.
   - After creating or updating a pull request, check the remote PR state before declaring the task complete.
   - Verify mergeability and review the CI/check-run status after the pushed head settles.
   - If the PR is not clean because of conflicts, failed CI, or another actionable issue, diagnose and resolve it, rerun relevant validation, and push the fix.
   - Stop and ask the user only when the blocker requires a user decision, credential, permission, reviewer action, protected-branch override, or external service recovery.
   - Do not merge the PR unless the user explicitly asks for that remote mutation.

## Personal Distribution Policy

- Do not rewrite imported skill bodies solely for house style, description format, progressive-loading style, or platform-neutral wording.
- Prefer preserving upstream `SKILL.md` content and placing local behavior in `AGENTS.md`, `docs/skill-routing.md`, `skills/workflow-router/`, owner workflow skills, and `capabilities/index.json`.
- Edit imported skill content only when the upstream text is unsafe, unusable in this repo, legally unclear, or directly conflicts with the routing overlay.
- Do not add host-specific tool names, config paths, UI metadata, or runner assumptions to distributed skill bodies.
- Do not add `agents/openai.yaml`, `.claude/skills/`, `.opencode/skills/`, or similar host-local metadata to the source skill tree.
- Keep project-local Codex bootstrap wrappers under ignored `.codex/skills/<skill-name>/`; each wrapper points back to the canonical `skills/<skill-name>/SKILL.md`.
- Put host packaging outside skills. Claude plugin support belongs in `.claude-plugin/`; the skill content remains standard.
- If an upstream skill assumes a specific runner, prefer routing notes, source records, or explicit-only status before rewriting its body.
- If a capability cannot be used safely without rewriting away its core value, keep it as an evaluated source or explicit-only reference.
- Keep repo harness templates under `harness/<template-name>/`; root harness files in target projects are installed only through explicit harness lifecycle commands, never through default skill installation.

## Codex Worktree Startup Gate

Before using repo-local skill activation or running `workflow-router` in a fresh Codex worktree, check whether `.codex/skills/` and `.harness-hub/state/` exist. Git worktree creation does not copy ignored generated directories.

For write tasks, run the tracked read-only preflight first:

```powershell
bun run codex:worktree-check -- --write-task
```

If the preflight fails, run the portable local setup before continuing:

```powershell
bun run codex:worktree-setup
```

For purely read-only questions, `bun run codex:worktree-check` may warn and still exit zero; continue only while clearly stating that repo-local skills are not yet bootstrapped.

This setup is local initialization only: it syncs `.codex/skills/`, creates missing `.harness-hub/state/` templates, preserves existing task state by default, and must not create a checkpoint commit. Use `bun run codex:worktree-setup -- --reset-state` only when intentionally starting over with clean local task state.

To avoid repeating this manual setup in this clone, install the advisory local Git hook once:

```powershell
bun run codex:worktree-hook:install
```

The hook uses Git's `post-checkout` hook, which also runs after `git worktree add` unless `--no-checkout` is used. It calls the same setup script, exits zero on setup failure, and does not perform remote writes.

## Skill Routing

Use `docs/skill-routing.md` to resolve overlapping skills. Prefer the narrowest matching skill:

- Non-trivial requests: use `workflow-router` first to classify the request into exactly one owner state; when a terminal is available, run or mirror `node skills/workflow-router/scripts/workflow-check.mjs --prompt "<request>" --json` before substantive work.
- Questions and evidence lookup: use `answer-workflow`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JasonxzWen/harness-hub](https://github.com/JasonxzWen/harness-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
