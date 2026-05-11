---
trigger: always_on
description: - Do not develop on `main` unless explicitly requested.
---

# Repository Workflow Rules (Lean)

## Branching
- Do not develop on `main` unless explicitly requested.
- For code changes, if the user does not specify a base branch, create a `codex/*` branch from the current branch.
- Codex branch names must start with `codex/`.

## Delivery Flow
- Only treat as full PR delivery when user explicitly asks (for example: "完成后提PR").
- Full delivery means: branch -> change -> local verify -> commit -> push -> PR -> follow CI -> merge when appropriate.
- If the user asks to keep `main` linear, rebase or otherwise update the working branch onto the latest `main` before opening or merging the PR, and use a merge method that adds a single linear commit on `main` without creating a merge commit. Prefer `squash merge` when the repository supports it; otherwise use `rebase merge`.
- Do not commit unless the user explicitly requests or approves commit for this task.
- If task is analysis/question only, do not create branch or commit.

## Exceptions
- If user says stay on current branch / work on `main` / skip PR / skip waiting CI, follow that instruction for this task.

## Swift & SDK Baseline
- Use Swift 6 for all Swift targets and new code.
- Keep deployment target at `15.6` unless user requests otherwise.
- Prefer modern APIs compatible with target `15.6`; avoid deprecated APIs.

## Compatibility Scope
- Default to the current supported interfaces and behavior.
- Do not preserve legacy interfaces, legacy behavior, compatibility layers, fallback branches, or duplicate paths unless user requirements, external callers, migration windows, or release plans explicitly require them.
- If legacy compatibility is retained, document the reason, removal condition, and validation impact in the handoff.

## Build Verification Gate
- After every code change, run local build verification.
- Handoff requires: zero compile errors and zero compile warnings.

## Test Execution Policy
- After every code change, explicitly check whether related tests need to be updated or added, and complete required test updates before handoff.
- Default: run targeted tests related to changed module/feature.
- If related verification has already completed after the latest code change, and no repo-tracked file has changed since that verification, a later commit-only instruction must reuse the existing fresh verification result instead of rerunning the same tests.
- For small, explicit, low-risk changes with tightly bounded impact, do not run the full `HomeSmokeTests` suite by default. Prefer build-only verification or a narrower targeted test that covers the changed control or flow.
- Run full suite when changes are broad/high-risk or impact cannot be bounded:
- shared/common code changes
- dependency/build settings/script/test infra changes
- large refactors (batch rename/signature/file moves)
- high-risk runtime behavior (concurrency/persistence/network/security)
- user explicitly requests full suite

## Test Permission Prompt Isolation
- Automated tests must not introduce product or app code paths that trigger avoidable macOS privacy prompts such as screen recording, microphone, camera, keyboard input, input method, or similar authorization dialogs.
- Any product code path that may request app privacy permissions must switch to a test-specific provider, mock, stub, or equivalent isolation layer under test environments.
- Test code must not rely on a human responding to app-driven privacy prompts, input method prompts, or avoidable local authorization dialogs to complete.
- macOS authorization required by the test harness itself, such as Automation, Accessibility, Input Monitoring, or related administrator approval for UI automation, is environment setup. UI tests may run when this setup is available.
- If a UI test fails, stalls, or times out because test harness authorization is missing or delayed, classify it as an environment setup failure and report it separately from product code or test code failures.
- Reject any test change that can block local or CI execution by introducing new avoidable privacy authorization prompts.

## UI Test Port Injection
- Preferred port key is `SharingPortPreferenceKeys.preferredPort` (`sharing.preferredPort`).
- For UI tests, inject with launch arguments: `-sharing.preferredPort <port>`.
- Do not write port value through hard-coded suite names in UI tests.

## AI Agent Temporary Workspace
- Put all AI-generated temp files/logs/artifacts under `.ai-tmp/`.
- Use isolated subdirectories under `.ai-tmp/`.
- Do not create ad-hoc temp dirs at repo root unless explicitly requested.

## AI Agent Plan Framing
- When the user asks for a plan, treat it as an execution plan for the AI agent unless the user explicitly assigns a human executor.
- Write plan steps from the agent's perspective.
- Do not frame the plan around human task management, personal schedules, or manual execution expectations unless the user explicitly asks for that format.
- If timing is needed, describe agent-relevant sequencing or wait states, such as build time, network latency, review gates, or external blocking conditions.
- If the user goal or instruction is ambiguous, do not guess. Ask for clarification promptly before continuing.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iamsyc/VoidDisplay](https://github.com/iamsyc/VoidDisplay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
