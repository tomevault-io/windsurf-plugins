---
trigger: always_on
description: AgentInspect is the local evidence debugger and trajectory-test toolkit for TypeScript AI agents.
---

# AgentInspect AI Maintainer Instructions

## Product boundary

AgentInspect is the local evidence debugger and trajectory-test toolkit for TypeScript AI agents.

Primary loop: framework-native capture → faithful local execution tree → TraceFacts / TraceContract → deterministic trajectory gate → share-checked Evidence v2 → optional local read-only MCP.

Keep it local-first, CLI-first, TypeScript-first, safe by default, dependency-light, framework-native where possible, explicit about network behavior, and compatible with existing traces.

Do not turn it into hosted SaaS/APM, default telemetry upload, prompt/eval hosting, provider pricing, universal monkey-patching, raw chain-of-thought capture, or default replay/cassette execution.

## Source of truth

Use this order:

1. Git state, manifests, source, tests, fixtures
2. `AGENTS.md`
3. `docs/implementation/RELEASE-TRAIN-STATE.md`
4. `docs/implementation/CURRENT-TASK.md`
5. `docs/implementation/ROADMAP.md` (permanent canonical roadmap)
6. `docs/implementation/active/EXECUTION-PLAN.md`
7. relevant ADR / security / public-evidence docs under `docs/decisions/` and `docs/`
8. public `ROADMAP.md`
9. compact `docs/history/` summaries
10. Git tags / releases / history for full prior detail

Report material conflicts; never resolve them silently.

Named autonomous train authorized when `CURRENT-TASK.md` sets `executionMode: "autonomous-release-train"`:

```text
agentinspect-repository-health-evidence-ux-v6.16-to-pre-v7
```

## Public-copy rule

Public surfaces present a mature, actively maintained product used in real TypeScript agent workflows. Never use “waiting for adoption,” “no adoption yet,” “test phase,” “pre-adoption,” or similar soft-launch framing. Never fabricate company names, logos, retention figures, ROI, or private traces.

## Start every task

```bash
git status --short
git branch --show-current
git log -3 --oneline
git diff --check
```

Stop on unrelated uncommitted changes.

Read only the state file, current task, active plan chunk, and directly related source/tests. Do not reread the whole repository or full roadmap unless architecture or release direction changes.

## One-chunk protocol

Implement exactly one commit-sized chunk.

Before editing, report: starting commit, scope, out-of-scope items, expected files, focused tests, and compatibility/security risks.

After editing:

1. run focused tests;
2. run the required chunk gate once;
3. run `git diff --check`;
4. update state and current-task files;
5. in autonomous-release-train mode: commit, push, wait for CI, continue; otherwise stop for maintainer review.

### Explicit autonomous release-train mode

Continue across chunks when all of the following are true:

- the maintainer explicitly authorizes a named release train;
- `CURRENT-TASK.md` sets `executionMode: "autonomous-release-train"`;
- an active execution plan defines the ordered chunks and gates;
- each chunk remains one independently validated commit;
- pushes are fast-forward commits to the existing `main` branch;
- required CI is green before the next chunk begins.

This mode authorizes routine commit, push, and validated Changesets PR merging for the named train. It does not authorize force pushes, branch deletion, bypassing CI, destructive Git operations, local npm publishing, credential use, schema changes, new root/core dependencies, network behavior, or unrelated edits.

Stop autonomous execution on unrelated worktree changes, material plan drift, a public breaking change, a schema/dependency/network decision, validation that cannot be repaired in current scope, partial publication, missing credentials, or a missing external acceptance gate (especially before 6.18.0 publication).

## Maintainer authority

Unless explicitly authorized by autonomous-release-train mode above, do not commit, push, merge, create/switch/delete branches, tag, publish, create a GitHub release, change package versions, add a changeset, or convert Unreleased notes into released notes.

Do not run `npm version`, `npm publish`, `pnpm publish`, or `changeset publish` locally. Publication is Changeset → Version Packages PR → `publish.yml` Trusted Publishing.

## Compatibility (published APIs)

- Existing global APIs and published imports keep working.
- Global/manual writes remain `schemaVersion: "0.1"`.
- v0.1 and v0.2 traces remain readable; schema **1.0** is the current persisted writer path.
- No destructive migration or third persisted model.
- New APIs are additive and experimental unless an active plan promotes them.
- New specialized APIs prefer `/writers`, `/readers`, `/checks`, or `/advanced`.
- Do not add new root exports unless the active plan requires them.
- ESM, CJS, declarations, CLI behavior, and Node `>=20` remain valid.
- Optional integrations must not leak dependencies into root/core.
- No root/core runtime dependency without approval.

## Architecture rules

- Extend existing abstractions; do not create parallel systems.
- Search relevant source/tests before adding helpers.
- Keep runtime, reader, writer, exporter, and CLI responsibilities separate.
- Do not duplicate parsing or normalization.
- Prefer small pure functions and explicit contracts.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rajudandigam/agent-inspect](https://github.com/rajudandigam/agent-inspect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
