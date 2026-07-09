---
trigger: always_on
description: always respond in Chinese unless the user explicitly asks for another language.
---

always respond in 中文
始终使用中文回复用户。
always respond in Chinese unless the user explicitly asks for another language.

## Project Local Rules

- Route interaction-reporting and handoff artifact work through `.codex/skills/effective-interact`.
- Run `corepack pnpm run validate` before final handoff when repository behavior or harness contracts change.

# Codex Harness

This repository is prepared for Codex-driven development. Treat tracked harness files as the stable rules, and use ignored local state under `.harness-hub/state/` for the active task, progress, decisions, and handoff.

## Initialization Gate

Do not start implementation until the harness is fully landed and the active task is goal-ready.

- Required files must exist: `AGENTS.md`, `feature_list.json`, `clean-state-checklist.md`, `definition-of-done.md`, `evaluator-rubric.md`, `quality-document.md`, `scripts/harness-validate.mjs`, `.harness-hub/state/{current-task.md,decisions.md,progress.md,session-handoff.md}`, and `.harness-hub/context/{AGENTS.md,README.md,llm-wiki-schema.md,wiki/index.md}`.
- If any required file is missing, stop and run the approved harness init path before coding.
- Run `harness-hub check . --json` as a read-only startup check when the command is available; if it is unavailable, use `npx @jasonwen/harness-hub@latest check . --json`. Treat CLI updates, target updates, missing locks, missing project-local Codex activation, and registry failures as advisory unless the current task explicitly requires updating.
- If startup check recommends `harness-hub activate-codex`, run the dry-run first and then confirm only when local Codex skill activation is needed: `harness-hub activate-codex . --dry-run --json`, then `harness-hub activate-codex . --yes`.
- Run `node scripts/harness-validate.mjs`; fix harness failures before changing product files.
- Fill `.harness-hub/state/current-task.md` with goal, assumptions, non-goals, allowed paths, forbidden paths, acceptance criteria, validation tiers, runtime signals, standard startup path, and checkpoint policy before editing.
- For feature, bug-fix, refactor, or product/spec change work, fill requirement intake, discovery/brainstorming, target spec, P0/P1/P2 test matrix, open questions, and alignment status before editing.
- Review recent progress, decisions, handoff, feature state, quality snapshot, branch/worktree, and `git status --short`.

## Operating Rules

- Keep this file short. Move detailed or historical context to task files, docs, or archives.
- Use `feature_list.json` as the stable feature and parallel-write policy inventory.
- Use the LLM Wiki under `.harness-hub/context/wiki/` as the stable knowledge middle layer. Raw sources remain authoritative; do not duplicate code facts or native agent memory. Follow `.harness-hub/context/AGENTS.md` and `.harness-hub/context/llm-wiki-schema.md`; write wiki knowledge only after human confirmation and record updates in `wiki/update-log.md`.
- Treat Loop Control Plane rules as the top-level automation boundary. Prompt, context, harness, and tool capabilities may be `standalone`, `composable`, or `loop-participant`; do not force standalone capabilities into fixed workflows.
- Keep workflow and Loop separate: workflow stages define how development moves from requirement intake to handoff; Loop only decides whether concrete capability actions continue or interrupt and records auditable decisions.
- Follow the Interrupt Policy in `.harness-hub/loop/policies/interrupt-policy.md`: continue low-risk local work when scope and validation are clear, but interrupt when risk signals require human review.
- Record auditable loop decisions in `.harness-hub/state/interrupt-decisions.jsonl`, loop runs in `.harness-hub/state/loop-runs.jsonl`, and capability events in `.harness-hub/state/capability-events.jsonl`.
- Use a separate git worktree or branch for each write task.
- Start from `.harness-hub/state/current-task.md` before changing files.
- Respect the task's allowed paths and forbidden paths.
- Treat Harness Hub source-repo packaging as non-target material: do not copy `.claude-plugin/`, root `openspec/`, `docs/`, `config/`, `package.json`, `README*`, source `AGENTS.md`, or this repo's source tree into target projects. Use the managed outputs from `harness-hub init-harness`, `install`, and `activate-codex` instead.
- Do not run parallel writes against the same file, module, or feature state.
- Use read-only parallel work only for research, review, log analysis, or validation.
- Use agentic loops for material planning, implementation, acceptance, PR closeout, and workflow-learning work when they reduce risk: Producer -> Verifier -> Arbiter -> Main Agent Decision.
- Keep loop roles host-neutral in repo state: `delegated-agent` may be a host-native subagent, isolated session, browser run, CI check, deterministic command, or bounded worker. Arbiters are read-only and must not edit files, resolve conflicts, push, publish, merge, or make final user-facing decisions.
- Do not let hooks auto-dispatch delegated agents. Hooks and deterministic checks may only remind, validate evidence, or interrupt for human review.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JasonxzWen/ai-daily-cn](https://github.com/JasonxzWen/ai-daily-cn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
