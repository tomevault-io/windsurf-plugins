---
trigger: always_on
description: This file is the repository router and cross-cutting governance layer for the active Expflow Build Week.
---

# AGENTS.md

## 1. Purpose

This file is the repository router and cross-cutting governance layer for the active Expflow Build Week.

It is intentionally compact. Detailed phase order, review cadence, merge rules, evidence requirements, and gate criteria belong in:

```text
docs/internal/BUILD_WEEK_WORKFLOW_CURRENT.md
```

Detailed implementation requirements belong in the active phase prompt.

The repository-local precision review procedure belongs in:

```text
.agents/skills/expflow-build-week-pr-review-precision/SKILL.md
```

Do not duplicate those documents here.

---

## 2. Source-of-truth order

Read and obey sources in this order:

1. root and applicable nested `AGENTS.md`;
2. immutable architecture, protocols, schemas, registries, canonical examples, compatibility contracts, and protected-surface declarations;
3. `docs/internal/BUILD_WEEK_WORKFLOW_CURRENT.md`; <!-- config-docref -->
4. `docs/internal/CURRENT_STATUS_MATRIX.md`; <!-- config-docref -->
5. `docs/internal/GLOSSARY.md`; <!-- config-docref -->
6. the active phase prompt under `docs/internal/phase_prompts/`;
7. repository-local skills required by the active phase;
8. accepted phase and gate reports;
9. relevant source, tests, CI, and package scripts;
10. external reviews and historical evidence as findings to reproduce;
11. conversation context as non-canonical working context.

A lower source may clarify a higher source. It must not override it.

When controlling sources conflict materially, stop the affected work and record the contradiction. Do not select whichever interpretation is easiest to implement.

`docs/releases/v1.1.0/` is frozen release provenance. It is not active implementation authority.

---

## 3. Active workflow

The active Build Week sequence is:

```text
BW-A
  Phase 1 — Ordinary UX/UI Corrections
  Phase 2 — Expflow GUI Foundation

BW-B
  Phase 3 — Stable Read Models
  Phase 4 — Evidence Intake and Authority Reconciliation
  Phase 5 — Portable Workflow Package

BW-C
  Phase 6 — Evidence-Backed Gap Closure
  Phase 7 — Pilot and Empirical Evaluation

BW-D
  Phase 8 — Guerilla Profile and Event Contracts
  Phase 9 — Guerilla Causal Event View GUI
```

Phase 1 is inherited as implemented but remains subject to the current repository status and accepted review record.

The current phase is determined by the workflow and repository evidence, not by an external launcher, conversation, branch name, or agent assertion.

No later phase may begin before the preceding phase is accepted and merged according to the workflow.

---

## 4. Product boundaries

### 4.1 Native authority

Native systems remain authoritative for native state.

- the filesystem is authoritative for current bytes within its scope;
- Git is authoritative for Git history and references;
- Reqtrace is authoritative for resolved traceability state;
- FIMP is authoritative for its mutation transaction and receipt;
- Expflow is authoritative for Expflow records and projections under its contracts;
- Guerilla is authoritative only for its event records and supported causal assertions.

Observation does not transfer authority.

Guerilla must not repair, replace, or silently reinterpret Expflow-native state.

### 4.2 Ordinary CLI

The ordinary Expflow command set remains:

```text
init
sync
status
restore
```

A fifth ordinary command requires an explicit architecture and compatibility decision.

Locked exit behavior:

- uninitialized `status`: exit `0`;
- operational mutation failure: exit `1`;
- usage failure and unknown command: exit `2`.

### 4.3 Restore

Restore remains:

- byte-exact;
- append-only;
- forward-committing;
- recoverable;
- non-destructive to recorded history;
- non-mutating during preview;
- refusing conflicting unrecorded drift by default;
- explicitly overridable only through the approved contract.

### 4.4 Identity

A provisional identity must be labeled provisional wherever it is exposed.

A provisional identifier must not be described as committed, stable, accepted, or durable.

### 4.5 GUI

The product name is **Expflow GUI**.

The approved root is:

```text
apps/gui/
```

The GUI must use documented application operations and read models. It must not treat raw undocumented `.expflow` storage as an application contract or create GUI-owned canonical state.

### 4.6 Compatibility

Do not silently break:

- v1 command automation;
- machine-readable output;
- persisted records;
- package exports;
- recovery behavior;
- public extension behavior;
- frozen release evidence.

Breaking changes require an explicit architecture and versioning decision.

---

## 5. Status and claims

Use these dimensions separately:

- implemented;
- internally verified;
- ordinary-surface available;
- phase accepted;
- gate accepted;
- pilot verified;
- empirically evaluated;
- production supported.

Do not replace them with a maturity percentage.

Do not claim:

- implementation from a schema, plan, prompt, mock, or screenshot;
- usability from internal tests alone;
- empirical success from reviewer approval;
- ordinary-surface availability from a library export;
- causal certainty from correlation;
- completion from material output alone.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paragon-ux/Expflow](https://github.com/paragon-ux/Expflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
