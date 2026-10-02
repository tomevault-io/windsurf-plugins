---
trigger: always_on
description: <!-- doc-locale: en -->
---

# CardputerZero-OS Agent Collaboration Rules

<!-- doc-locale: en -->
> **English** | [简体中文](AGENTS.zh-CN.md)

This file applies to the entire repository. It defines durable rules for
collaboration, implementation, and validation; it does not record temporary
progress for a particular task. If a subdirectory gains a more specific
`AGENTS.md`, that file applies only within the subdirectory and must not relax
the security, requirement, or acceptance boundaries defined here.

## 1. Sources of Truth

Use the following order when interpreting requirements and deciding whether an
implementation is correct:

1. The user's current explicit requirements, constraints, and latest feedback.
2. Open acceptance criteria in `docs/ROADMAP.md` and related specialized
   roadmaps.
3. Architecture decisions with Accepted status in `docs/adr/`.
4. `docs/ARCHITECTURE.md`, `docs/THREAT-MODEL.md`, and the corresponding phase
   documents.
5. Existing contracts expressed by committed code, schemas, tests, and build
   scripts.
6. Supporting material such as README files, historical reports, and comments.

When these sources conflict, do not select the most convenient interpretation.
First determine whether a source is obsolete, then clearly report the conflict,
impact, and recommendation to the primary agent or the user. Evidence from a
physical device takes precedence over inferences from code that has not been
validated on a physical device, but every observation must still record the
image version, hardware version, and reproduction conditions.

Do not treat the following as completed requirements: unchecked roadmap items,
future approaches in ADRs, test fixtures, mocks, hardware paths that pass only
on the host, or work explicitly marked pending, deferred, or open in the
documentation.

## 2. Synchronization Before Work

Before analysis or editing, every agent must:

- Read this file and any more specific `AGENTS.md` in the target directory.
- Read the roadmaps, ADRs, phase documents, and tests directly related to the
  task.
- Run `git status --short` to identify existing modifications and untracked
  files.
- Run `git diff -- <path>` for every file it plans to edit, comparing the
  working copy with `HEAD`.
- Check for other consumers of the same behavior, including image builds,
  runtimes, services, schemas, tests, and documentation.
- Work from the current workspace contents, not from a stale snapshot captured
  when the task began.

Existing modifications belong to the user or another agent by default. Do not
revert, overwrite, reformat, or opportunistically clean up unrelated changes.
When existing changes overlap the task, understand and build on them; request
coordination only when they cannot be merged safely.

## 3. Multi-Agent Collaboration Protocol

The primary agent is responsible for interpreting requirements, dividing work,
final integration, and reporting the result. A sub-agent handles only its
explicitly assigned scope and must not expand requirements or modify adjacent
modules without authorization.

### 3.1 Division of Work

- Prefer assigning edit ownership by non-overlapping directories or files. A
  shared file may have only one writer at a time.
- Research, code reading, and test analysis may run in parallel; continuous
  edits along one implementation path should be owned by one agent.
- Every assignment must state the objective, requirement basis, paths that may
  be modified, paths that must not be touched, and expected validation.
- A sub-agent that discovers an out-of-scope issue reports its evidence,
  severity, and recommendation without editing it.
- The primary agent must recheck workspace state both when dispatching work and
  when receiving a handoff, so it does not rely on stale information.

### 3.2 State Synchronization

Task state comes from current user messages, agent messages, and the actual
workspace. Do not infer current state solely from a conversation summary, old
diff, previous test result, or this file.

Every handoff must contain the following fields. If a field cannot be completed,
state why:

```text
Objective:
Status: complete / partially complete / blocked
Requirement basis:
Key decisions:
Modified files:
Related files not modified:
Validation performed: command + PASS/FAIL
Validation not performed: reason
Open risks or physical-device retest items:
```

Test results must match the latest files at handoff time. If any edit occurs
after a test, that earlier result must not be reported as the final PASS. After
integrating all work, the primary agent must reread the complete diff and rerun
validation appropriate to the final state.

### 3.3 Conflict Handling

- If a file changes during the work, stop writing it and reread the diff before
  deciding how to merge.
- Do not use `git reset --hard`, `git checkout --`, forced cleanup, or bulk
  overwrites to resolve collaboration conflicts.
- Do not delete files of unknown origin or claim another agent's modifications
  as your own validated work.
- Changes to a shared interface require coordinated compatibility checks across
  producers, consumers, schemas, versions, and test fixtures.

## 4. Requirement-Driven Change Boundaries

Every code change must trace to a user requirement, roadmap acceptance

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imliubo/cp0-os-experiment](https://github.com/imliubo/cp0-os-experiment) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
