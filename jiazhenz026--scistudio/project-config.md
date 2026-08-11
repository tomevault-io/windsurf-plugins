---
trigger: always_on
description: This is the root behavior guide for SciStudio repository work.
---

# AGENTS.md

This is the root behavior guide for SciStudio repository work.

Runtime files, skills, memory, and tool-specific configs must point here and to
canonical docs instead of creating separate policy.

## 1. Project Identity

SciStudio is an AI-native workflow runtime for multimodal scientific data. It uses
typed workflow graphs, formal block contracts, persisted artifacts,
plugin-based extension, manual review steps, and AI-assisted orchestration.

## 2. How To Work Here

- Start by understanding the owner request, issue, existing docs, and affected
  code before editing.

- Choose the correct task kind and persona before work starts.
  Use the document index below for routing.

- Keep the task small, traceable, and reviewable.
  Do not silently expand scope.

- Prefer existing contracts, helpers, patterns, and docs over new local
  inventions.

- When unsure, preserve architecture, traceability, and small scope first.
  State assumptions instead of hiding them.

- Treat tests, docs, gate evidence, and CI as part of the work, not afterthoughts.

## 3. Required Rules

### 3.1 Development Hard Rules

- Work on a dedicated branch and worktree.
  Do not share a writable worktree across agents.

- Do not use `pip install -e .`.
  It can pollute the shared environment.

- Every PR must close an open issue.
  Do not create a new issue when an open issue already tracks the work.

- AI-authored work must use the committed gate-record workflow.
  Chat messages and local-only notes are not gate evidence.

- Tests are part of implementation work.
  Add or update tests when behavior changes.

- Documentation is part of the change.
  Update docs or record a clear N/A reason.

- Do not bypass schemas, lineage, runtime checks, governance checks, or CI.

- Do not weaken governance, CI, Sentrux, quality thresholds, or protected paths
  unless the owner explicitly approved that scope.

- CI must pass before work is complete.

### 3.2 Traceability

- Meaningful work must follow:
  Idea -> Issue -> Spec/ADR when needed -> Branch -> Commit -> PR -> Review -> CI/Test -> Merge -> Release.

- Use focused branches, focused commits, and focused PRs.
  Do not mix unrelated work.

- Do not push directly to protected branches.
  Do not merge PRs without explicit owner or administrator authorization.

### 3.3 Branch And Worktree Rules

- Use a dedicated branch for each task.

- Use a dedicated worktree for AI-authored work and parallel work.

- Do not share a writable worktree across agents.

- Do not merge local work into `main`.

### 3.4 Specs, ADRs, And Docs

- Update a spec when work changes contracts, schemas, runtime behavior, storage,
  API behavior, plugin contracts, UI semantics, AI orchestration, or external
  app integration.

- Update an ADR when the change is architectural, hard to reverse,
  cross-module, likely to be questioned later, or a long-term tradeoff.

- Documentation is part of the product.
  Update docs when behavior, workflow, public contracts, or architecture change.

- Generated docs must stay generated.

### 3.5 Coding Boundaries

- Prefer explicit contracts over clever shortcuts.

- Favor composition over deep inheritance.

- Keep modules narrow in responsibility.

- Do not move plugin logic into core for convenience.

- Do not place runtime truth in frontend state.

- Do not bypass schemas, lineage, runtime checks, governance checks, or CI.

### 3.6 Deferred Work

- Deferred work must be visible in the repository.

- Any out-of-scope behavior, edge case, cleanup, test, or follow-up must use a
  tracked TODO that cites an issue, ADR, spec, PR, or follow-up ticket.

```python
# TODO(#NNN): <what is deferred and why>
#   Out of scope per <ADR/spec/PR/owner decision>.
#   Followup: <issue URL or tracking reference>.
```

- Untracked "later", "MVP", "V1", or chat-only deferrals are not acceptable.

### 3.7 AI Agent Rules

- AI agents must read `docs/ai-developer/rules.md`.

- AI agents must use the matching task rule in
  `docs/ai-developer/specific_rules/`.

- AI agents must use the matching persona in `docs/ai-developer/personas/`.

- AI-authored work must use
  `docs/ai-developer/specific_rules/gated-workflow.md`.

- Supported task kinds: `hotfix`, `bugfix`, `feature`, `refactor`, `docs`,
  `maintenance`, `manager`, `guided`. The `guided` task kind is for
  owner-directed live implementation sessions; its default persona is
  `live_implementer` and its specific rule is
  `docs/ai-developer/specific_rules/guided-work.md`.

- Supported personas: `manager`, `implementer`, `adr_author`,
  `audit_reviewer`, `test_engineer`, `live_implementer`. The
  `live_implementer` persona guide is at
  `docs/ai-developer/personas/live-implementer.md`.

- Gate records must be created and updated with the gate record ledger CLI:
  `python -m scistudio.qa.governance.gate_record`.
  The gate record is the single source of truth (ADR-042 Addendum 6).
  Receipt behavior is folded into the ledger. There is no separate
  `gate_receipt` command; use `gate_record check --mode pre-pr` and
  `gate_record finalize` instead.

- The primary workflow commands are `init`, `plan`, `amend`, `check`, and
  `finalize`. `check` observes the git diff, infers tier-selected

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jiazhenz026/SciStudio](https://github.com/jiazhenz026/SciStudio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
