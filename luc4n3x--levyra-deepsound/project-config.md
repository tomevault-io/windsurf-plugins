---
trigger: always_on
description: This file is the repository-wide operating contract for coding agents. Claude
---

# Levyra Engineering Instructions

## Purpose and hierarchy

This file is the repository-wide operating contract for coding agents. Claude
Code, Codex, ChatGPT when operating through a repository-capable runtime,
Google Antigravity, OpenCode, OpenClaw, and compatible runtimes should read it
from the Git root, then apply any nearer `AGENTS.md` for the files in scope.

Instruction order:

1. root `AGENTS.md`;
2. nearer path-specific `AGENTS.md` files;
3. approved requirements and active planning in `docs/project/SPEC.md`,
   `docs/project/ROADMAP.md`, and `docs/project/TASKS.md`;
4. matching native skills under `.agents/skills/`;
5. current architecture, implementation, tests, build files, and workflows;
6. detailed Levyra playbooks under `.claude/skills/` and `.claude/rules/`.

Current repository evidence always overrides remembered behavior, old
discussions, stale comments, previous agent output, or stale task status.
Surface conflicts between planning files and implementation before editing.

For production-code implementation or broad review, also read
`docs/ai/AI_ENGINEERING_GUARDRAILS.md` and apply its reuse-first, explicit
assumption/tradeoff, simpler-alternative, goal-verification, surgical-edit,
complexity-budget, and diff-quality rules.

## Repository map

- `docs/README.md`: canonical documentation index.
- `docs/project/SPEC.md`: durable product and engineering requirements.
- `docs/project/ROADMAP.md`: ordered outcomes, risks, and phase exit criteria.
- `docs/project/TASKS.md`: one active reviewable phase and its truthful
  validation state.
- `app/`: Android client; additional rules in `app/AGENTS.md`.
- `desktop/`: independent Windows client; additional rules in
  `desktop/AGENTS.md`.
- `.github/`: CI and release automation; additional rules in
  `.github/AGENTS.md`.
- `docs/`: project documentation; additional rules in `docs/AGENTS.md`.
- `docs/project/`: product specification, engineering roadmap, and active task
  phase.
- `docs/agents/`: repository-specific configuration consumed by external agent
  skills, including issue-tracker and domain-document conventions.
- `docs/ai/`: ChatGPT, Codex, Google Antigravity, Claude Code, and OpenClaw
  collaboration guidance.
- `.agents/rules/`: lightweight workspace rules that link to this canonical
  contract.
- `.agents/skills/`: native Codex, Antigravity, OpenAI, and OpenClaw-compatible
  skills.
- `.claude/`: Claude Code configuration plus reusable Levyra engineering
  playbooks.

## Agent runtime discovery

- Root `AGENTS.md` is the canonical repository context for supported coding
  agents.
- Google Antigravity discovers workspace skills from
  `.agents/skills/<skill-name>/SKILL.md` when a conversation starts.
- `.agents/rules/levyra-workspace.md` links back to this file with a relative
  `@../../AGENTS.md` reference instead of maintaining a duplicate contract.
- Open the repository root as the workspace. Starting only from `app/`,
  `desktop/`, or another nested folder may hide repository-level agent
  configuration.
- Start a new agent conversation after pulling instruction or skill changes so
  the runtime can rebuild its workspace inventory.
- See `docs/ai/ANTIGRAVITY.md` for Antigravity setup and verification.

Runtime-specific discovery must not create separate sources of truth. This file,
nearest scoped instructions, approved planning, matching skills, and current
repository evidence remain the shared hierarchy.

## Execution-first operating mode

This policy is shared by every supported coding runtime. It is not a
Claude-specific preference.

- When the owner asks to `fix`, `update`, `address`, `implement`, `refactor`, or
  otherwise change code, execute the requested work directly inside the
  authorized scope. Do not turn an implementation request into an unsolicited
  design discussion, tutorial, list of alternatives, or approval pause.
- Do not ask routine confirmation questions when the requested outcome,
  repository evidence, and existing architecture already determine the next
  safe action.
- Ask only when an indispensable input is missing, materially different
  interpretations would produce meaningfully different results, or the next
  action is destructive, irreversible, security-sensitive, or outside the
  owner's existing authorization.
- If the requested approach conflicts with current repository evidence,
  security, data integrity, compatibility, product invariants, or deterministic
  validation, surface the conflict briefly and choose the smallest safe
  correction when it remains inside scope. Never silently execute a known-bad
  approach merely to appear compliant.
- For code-bearing work: inspect the current implementation and nearby tests, make the
  smallest coherent change, run focused validation, fix regressions introduced
  by the change, inspect the final diff, then report the result, files changed,
  checks run, and any real blocker.
- Treat `only this`, `solo questo`, and equivalent wording as a hard scope
  boundary. Do not perform opportunistic refactors, unrelated cleanup,
  dependency churn, version changes, or speculative architecture work.
- Prefer execution over commentary. Explain decisions only when they materially

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LUC4N3X/Levyra-deepsound](https://github.com/LUC4N3X/Levyra-deepsound) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
