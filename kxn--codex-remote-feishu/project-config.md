---
trigger: always_on
description: - For direct user instructions, first send a short restatement:
---

# AGENTS

## Conversation Handshake (Always On)

- For direct user instructions, first send a short restatement:
  - what the user asked
  - what you will do immediately
- Then execute without waiting unless user explicitly asks to pause.
- If user gives correction/steer feedback, switch direction immediately.

## Trigger Accuracy Rule (Always On)

- Skill triggering uses **union matching**: trigger when **any** of these match:
  - user wording / command intent
  - touched logic carrier
  - touched file area
  - known symptom pattern
- Do not narrow trigger scope below prior behavior.
- If multiple skills match, use all relevant skills together.
- Exclusion notes (for example “pure copy/styling/logging/tests only”) apply only when you can confirm logic carriers are unchanged.

## Workspace Cleanliness Rule

For every new repository task in chat (not only GitHub issue workflow):

- Before starting substantive read/edit/build work, check current workspace cleanliness with `git status --short`.
- If the worktree is clean, proceed normally.
- If the worktree is not clean, do not silently continue with mixed context:
  - first classify existing local changes as either `same-task` or `different-task`
  - if `different-task`, stop and ask the user whether to:
    - commit/push them first
    - shelve them
    - or explicitly continue in dirty workspace
  - if `same-task`, explicitly state that assumption in chat and continue
- Do not mix unrelated edits into one commit by default.
- When the user asks to "先提交" or similar, complete that commit before starting additional implementation work.

## Deterministic Repo Helper Rule

- For repeated tail-state or publish-state checks, prefer `bash scripts/dev/worktree-facts.sh` instead of manually rerunning `git status --short --branch`.
- Before opening a guessed repo path, prefer `bash scripts/dev/resolve-repo-path.sh <path>` or `rg --files` so path probes do not fail by typo.
- Before using unfamiliar `gh ... --json` fields, prefer `bash scripts/dev/gh-json-fields.sh <gh-subcommand...>` and `--check ...` when needed.
- Do not rerun an identical deterministic failure unless some input, state, or environment has changed; first state what changed.

## Staged Execution Continuity Rule

When the user explicitly asks staged rollout (for example: `按阶段推进`, `分阶段推进`, `阶段式推进`, `staged rollout`):

- Treat it as continuous execution by default.
- Complete all planned stages in one flow unless a real blocker appears.
- Do not pause after one stage to ask whether to continue.
- Treat a phase or stage as execution order only, not as a normal stopping boundary.
- For unsplit issue work, finishing `阶段 A / B / C` is never by itself a valid reason to stop.
- At the end of every phase, explicitly re-check:
  - is the issue already fully complete?
  - is there a hard blocker or real contradiction?
  - did execution hit a product decision gate?
  - does the issue now need a formal split before continuing?
- If all answers above are no, continue immediately into the next phase instead of stopping at the phase boundary.
- For repository or issue work, do not treat “local implementation is done” as a valid stopping point by itself.
- The staged flow is only complete when the normal tail work is also done for that task:
  - validation finished
  - required docs / issue state synced
  - commit completed when the change is worth keeping
  - push completed when repo policy says it should be pushed
  - issue closed when the issue is actually finished
- Valid stop conditions:
  - hard blocker (dependency/outage/permission)
  - newly discovered contradiction that makes continuing unsafe
  - explicit user redirection
- On stop, report blocker evidence and exact resume action.

## Skill Trigger Matrix

### `relay-stack-playbook`

Use `.codex/skills/relay-stack-playbook/` when working on:

- `relayd` / `relay-wrapper`
- Feishu bot inbound/outbound behavior
- VS Code remote integration
- Codex app-server protocol translation
- `/list`, `/attach`, `/use`, `/stop`
- queue/dispatch/thread routing/surface session issues
- helper/internal traffic classification issues
- “VS Code 有回复但飞书没回复” and similar missing-reply symptoms

### `remote-state-machine-guardrail`

Use `.codex/skills/remote-state-machine-guardrail/` for remote-surface state-machine logic carriers:

- attach/detach, `/use`, `/follow`, `/new` state predicates or transitions
- selected-thread / attached-instance / input-routing decisions
- queue routing / dispatch mode / pause-handoff gating
- headless launch/resume/cancel/timeout/recovery progression
- request-capture / prompt-gate / modal-gate / staged-input / selection-flow enter-exit
- command-availability matrix
- any change that adds/removes remote surface states

Do not trigger only for pure copy/styling/logging/tests/refactor with no logic-carrier change.

### `feishu-ui-state-machine-guardrail`

Use `.codex/skills/feishu-ui-state-machine-guardrail/` for Feishu card UI state-machine logic carriers:

- callback payload schema/parsing
- card owner/kind/action routing
- inline replace vs append-only decision
- command menu / selection prompt / request prompt navigation
- `daemon_lifecycle_id`, old-card rejection, freshness/lifecycle stamping

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kxn/codex-remote-feishu](https://github.com/kxn/codex-remote-feishu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-27 -->
