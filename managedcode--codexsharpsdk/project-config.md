---
trigger: always_on
description: Project: ManagedCode.CodexSharpSDK
---

# AGENTS.md

Project: ManagedCode.CodexSharpSDK
Stack: .NET 10, C# 14, TUnit, GitHub Actions, NativeAOT, NuGet, Codex CLI integration

Follows [MCAF](https://mcaf.managed-code.com/)

---

## Conversations (Self-Learning)

Learn the user's habits, preferences, and working style. Extract rules from conversations, save to "## Rules to follow", and generate code according to the user's personal rules.

**Update requirement (core mechanism):**

Before doing ANY task, evaluate the latest user message.
If you detect a new rule, correction, preference, or change -> update `AGENTS.md` first.
Only after updating the file you may produce the task output.
If no new rule is detected -> do not update the file.

**When to extract rules:**

- prohibition words (never, don't, stop, avoid) or similar -> add NEVER rule
- requirement words (always, must, make sure, should) or similar -> add ALWAYS rule
- memory words (remember, keep in mind, note that) or similar -> add rule
- process words (the process is, the workflow is, we do it like) or similar -> add to workflow
- future words (from now on, going forward) or similar -> add permanent rule

**Preferences -> add to Preferences section:**

- positive (I like, I prefer, this is better) or similar -> Likes
- negative (I don't like, I hate, this is bad) or similar -> Dislikes
- comparison (prefer X over Y, use X instead of Y) or similar -> preference rule

**Corrections -> update or add rule:**

- error indication (this is wrong, incorrect, broken) or similar -> fix and add rule
- repetition frustration (don't do this again, you ignored, you missed) or similar -> emphatic rule
- manual fixes by user -> extract what changed and why

**Strong signal (add IMMEDIATELY):**

- swearing, frustration, anger, sarcasm -> critical rule
- ALL CAPS, excessive punctuation (!!!, ???) -> high priority
- same mistake twice -> permanent emphatic rule
- user undoes your changes -> understand why, prevent

**Ignore (do NOT add):**

- temporary scope (only for now, just this time, for this task) or similar
- one-off exceptions
- context-specific instructions for current task only

**Rule format:**

- One instruction per bullet
- Tie to category (Testing, Code, Docs, etc.)
- Capture WHY, not just what
- Remove obsolete rules when superseded

---

## Rules to follow (Mandatory, no exceptions)

### Commands

- build: `dotnet build ManagedCode.CodexSharpSDK.slnx -c Release -warnaserror`
- test: `dotnet test --solution ManagedCode.CodexSharpSDK.slnx -c Release`
- format: `dotnet format ManagedCode.CodexSharpSDK.slnx`
- analyze: `dotnet build ManagedCode.CodexSharpSDK.slnx -c Release -warnaserror /p:TreatWarningsAsErrors=true`
- coverage: `dotnet test --solution ManagedCode.CodexSharpSDK.slnx -c Release -- --coverage --coverage-output-format cobertura --coverage-output coverage.cobertura.xml`
- For Codex CLI metadata checks (for example model list/default), always use the installed `codex` CLI directly first; do not use `cargo`/submodule helper binaries unless explicitly requested.

### Task Delivery (ALL TASKS)

- Always start from `docs/Architecture/Overview.md`:
  - identify impacted modules and boundaries
  - identify entry points and contracts
  - follow linked ADR/Feature docs before code changes
- Keep scope explicit before coding:
  - in scope
  - out of scope
- Keep context minimal and relevant; do not scan the whole repository unless required.
- Update docs for every behavior or architecture change:
  - `docs/Features/*` for behavior
  - `docs/ADR/*` for design/architecture decisions
  - `docs/Architecture/Overview.md` when module boundaries or interactions change
- Implement code and tests together.
- When asked to fix review findings, close every confirmed finding in the same pass; do not leave partial fixes.
- Do not keep or add public sample projects; repository focus is SDK + tests only.
- Upstream sync automation must track real `openai/codex` CLI changes (flags/models/features), not TypeScript SDK surface diffs, and open actionable repository issues for required SDK follow-up.
- Automatically opened upstream sync issues must include change summary/checklist and stay unassigned unless the user explicitly requests an assignee.
- For `openai/codex` repo sync/update work, always inspect the bundled `models.json` catalog in `submodules/openai-codex` (prefer `codex-rs/models-manager/models.json`, fall back to `codex-rs/core/models.json` for older pins) and reconcile SDK model constants against that repo-authoritative source.
- At the end of implementation/code-change tasks, create a git commit unless the user explicitly says not to, so the workspace ends in a reviewable state.
- Run verification in this order:
  - focused tests for changed behavior
  - full solution tests
  - format
  - final build
- Do not bump package/release version for test-only changes, including PRs driven by submodule sync, when they do not modify the repository's shipped SDK code or consumer-facing behavior; commit those changes without a release to avoid unnecessary package churn.
- If changes impact trimming/AOT safety, validate via AOT-safe API design (for example `JsonTypeInfo<T>` overloads and annotations) and existing test/build gates; do not introduce a dedicated AOT smoke test project unless explicitly requested.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [managedcode/CodexSharpSDK](https://github.com/managedcode/CodexSharpSDK) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
