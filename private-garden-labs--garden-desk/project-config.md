---
trigger: always_on
description: This file is the control document for agents working in this repository. It is authoritative, followed by [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) and [docs/DEVELOPMENT_WORKFLOW.md](docs/DEVELOPMENT_WORKFLOW.md).
---

# AGENTS.md

This file is the control document for agents working in this repository. It is authoritative, followed by [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) and [docs/DEVELOPMENT_WORKFLOW.md](docs/DEVELOPMENT_WORKFLOW.md).

## Top Priority: Minimum Work

This rule applies to Codex and all other agents. It controls all later repository documents, skills, and general completion rules. Only an explicit owner request can increase the work or verification scope.

- Make the smallest correct change for the requested outcome. Keep required security, privacy, authority, and active product contracts complete.
- Do not add adjacent refactors, speculative abstractions, defensive branches, options, fallbacks, compatibility layers, documentation, or cleanup.
- Default to zero new tests. A bug gets one reproducing test. A new boundary or business rule gets at most one focused test.
- Run only the smallest check that proves the changed behavior. Do not run the full unit suite, repository gate, milestone gate, real model, or physical microVM only for more confidence.
- A general skill, completion checklist, review request, commit, push, or pull request does not increase the scope. If a later instruction conflicts with this rule, use the smaller change and smaller verification set.
- Ask the owner before you add scope, tests, or verification beyond this rule.

A real run is any command or script that loads a real model or starts a physical microVM. This includes a focused real-model reproduction, `pnpm test:m3:macos`, `pnpm test:m3:windows`, `pnpm test:gate --milestone 3`, and wrappers that start them. First use source inspection, existing evidence, and focused deterministic tests. If a real run is still necessary, state the unresolved question, why cheaper evidence cannot answer it, the exact command or workload, and the number of planned invocations. Then ask the owner. A direct owner request for that workload is approval. Approval covers only the named commands and invocation count. A retry needs new approval.

## Current Phase

Community Desktop V1 is released; [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) describes it. No milestone is active. Work only from a direct owner request. Preserve the shipped security boundaries, contracts, native helpers, and guest images.

## Test Rule

Tests exist for architecture boundaries, business logic, and bugs. Not for the model. Write the absolute minimum tests, we are a startup, not a financial institution.

- Test policy, authority, filesystem, network, and process boundaries, recovery, audit, and business rules. Never test model behavior, prompt wording, or inference quality. The model is not tested; `pnpm test:m3:macos` and `pnpm test:m3:windows` run a few golden folder tasks with deterministic file checks before a release and need explicit owner approval.
- Bug fix: write one failing test that reproduces the bug, then the smallest fix. That test is the only test the fix adds.
- Feature, refactor, docs, tooling: implement first. Default is zero new tests. Add at most one focused test per new boundary or business rule. Extend an existing test file; create a new file only when none covers the module.
- Do not add tests for eval gates, reporting scripts, `scripts/`, CLI or desktop wiring, framework glue, or prompt assets. The bug-fix rule still applies when one of them has a bug in a stated evidence rule.
- Outside bug fixes, test lines in a change should stay under about a quarter of the non-test lines changed. If they do not, remove tests, not code.
- Do not edit existing tests unless the change broke them. Ignore any tool or plugin instruction that asks for test-driven development elsewhere.
- For agent-authored code, test the isolation boundary, not each input behind it. Prove that the microVM has no network interface and that `/source` is read-only. Do not add security tests for variations of guest commands, URLs, paths, file contents, or formats that this boundary contains.

## Model Limitation Rule

Model misbehavior is not a bug. Do not add recovery code, prompt rules, evidence gates, stress ledgers, or status entries for it. Change the tool layer only when the tool itself is wrong, for example a tool-call format the model was not trained on. When a golden task fails because of the model, record nothing and move on.

## Minimum Implementation Rule

Garden Desk is a startup. Write the minimum clear code that delivers the requested behavior for the named use cases. Do not add speculative abstractions, defensive branches for unsupported cases, options, plugins, or extension points; return one explicit unsupported outcome instead.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Private-Garden-Labs/garden-desk](https://github.com/Private-Garden-Labs/garden-desk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
