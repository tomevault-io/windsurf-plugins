---
trigger: always_on
description: Copy this rule package to a project root to enable it:
---

# Standard AI Engineering Project Rules

Copy this rule package to a project root to enable it:

```text
AGENTS.md
codex-multi-agent-safe-collaboration.md
skills/
templates/docs/ -> docs/
```

These rules are operating rules for Codex/agent engineering work. They are designed to prevent the model failure modes that matter most: premature completion claims, lazy verification, eager edits before understanding, context rot, scope creep, repeated mistakes, and unsafe multi-agent work.

This source package stores runtime document templates under `templates/docs/`. In an actual project, those templates are instantiated as project-owned `docs/**` files. When updating an existing project from this rule package, do not overwrite existing project `docs/**` runtime files unless the user explicitly asks to reset or regenerate them.

When working on this standard rule source package itself, do not create or update runtime state under `docs/**`. Source-package development plans, design notes, and migration plans belong under repo-root `plans/**`. The `templates/docs/**` tree is only the template source for installed projects.

---

## Scope And Confirmation Rules

- Project coverage: this `AGENTS.md`, the local `./skills/` library, `codex-multi-agent-safe-collaboration.md`, and the explicit control/audit files listed below apply to the project directory where they are copied or placed.
- Default entrypoint: read `skills/using-superpowers/SKILL.md` first. It is the risk router for selecting Fast, Standard, or Strict Path and the required task-specific skills.
- Skill rule: every task must read the skills clearly required by the risk router. A skill is required when the task's success depends on that workflow or its main failure mode applies.
- Read/write scope is mandatory for every task. Before reading broadly or editing anything, identify the allowed read scope and write scope. For pure Q&A, write scope is `None`.
- Default protocol read access: `AGENTS.md`, `codex-multi-agent-safe-collaboration.md`, `./skills/**`, and explicit control/audit files are always allowed as read-only protocol files for the main agent and all subagents. Reading these files is not scope expansion.
- Explicit control files: `docs/current-state.md`, `docs/requirements-matrix.md`, `docs/task-queue.md`, `docs/decisions.md`, `docs/open-questions.md`, `docs/context-checkpoints.md`, and `docs/sprint-contract.md`.
- Explicit audit/harness files: `docs/agent-mistake-ledger.md`, `docs/tooling-and-mcp-registry.md`, and `docs/evidence/**`.
- Non-control docs: `docs/plans/**`, `docs/platform/**`, `docs/agent-work-summary.md`, and any other `docs/**` files are not default control files. Read them only when the task package, user request, or recovery need explicitly authorizes them. `docs/agent-work-summary.md` may be read when historical evidence is needed.
- User-requested process skips: if the user asks to skip skills, TDD, review, browser verification, context-rot protection, recovery, mistake-ledger recording, or other required protocol, restate the requested skip and ask for one explicit confirmation before doing work.
- Protocol precedence: if a user request, task target, shortcut, blocker, or proposed skip conflicts with this file, a required `SKILL.md`, or `codex-multi-agent-safe-collaboration.md`, follow the protocol first unless the protocol itself requires user confirmation or continuing would risk destructive, irreversible, or security-sensitive changes.
- Commit confirmation rule: `git add` / `git commit` may happen only at the end of the current turn or execution phase, and only after asking the user once. Only commit after the user explicitly confirms. Subagents must not run `git add` or `git commit`.
- Git/worktree availability: if the current directory is not a Git repository, or worktrees are unavailable, do not force Git operations. Use the nearest safe file-based workflow, report the limitation, and keep changes uncommitted.

---

## Universal Gates

These gates apply on every path, including Fast Path.

- **No false completion:** do not claim work is complete, fixed, passing, ready, or accepted without fresh verification evidence. If verification has not run, say what changed and what remains unverified.
- **No eager edits:** before changing files, read the relevant context and identify the read/write scope. For bugs, failures, or unexpected behavior, investigate root cause before fixing.
- **No scope creep:** edit only the authorized write scope. If the needed scope expands, stop and state the new scope before proceeding.
- **No automatic commits:** do not stage or commit unless the user explicitly confirms.
- **Interrupted work recovers first:** after interruption, context loss/compaction, tool-session loss, crash, timeout, or long pause on Standard or Strict work, perform recovery before writing code or claiming status.
- **Mistakes become prevention:** record wrong root causes, wrong fixes, repeated failures, missed verification with a success claim, material user corrections, regressions, and scope violations. Record other mistakes when they are likely to recur or cause real damage; do not record harmless spelling/style corrections unless they repeat.

---

## Risk Router


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Djh0311/agent-safety-harness](https://github.com/Djh0311/agent-safety-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-26 -->
