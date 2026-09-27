---
trigger: always_on
description: This file applies to the whole repository. Keep work correct, narrow, reversible, and fast. A nested `AGENTS.md` may add path-specific rules but may not weaken the safety, scope, or rollback rules here.
---

# WebHTV Agent Contract

This file applies to the whole repository. Keep work correct, narrow, reversible, and fast. A nested `AGENTS.md` may add path-specific rules but may not weaken the safety, scope, or rollback rules here.

## 1. Start with a bounded lane

Before the first edit, state one completion sentence, the allowed paths, protected pre-existing dirty paths, the current local time and timezone, a realistic total-duration estimate with expected finish time, and the cheapest decisive verification. For multi-phase work, give a short estimate for each phase. Treat these estimates as execution targets rather than guard gates; when the estimate is reached or slips materially, stop optional work, state the cause, and continue with the narrowest completion path or a materially different shortest route. Do not let a stale estimate justify repeated checks, open-ended research, or an unfinished handoff. Use the smallest applicable lane:

Estimate elapsed wall-clock time for the current Codex agent in this workspace, not the time a human engineer or team would need. Base the estimate on the actual repository state, available tools, warm caches, ABI/build scope, device availability, and unavoidable network/build/user wait; split those phases when they materially differ and give the expected local finish time. Do not quote person-days or staffing estimates unless the user explicitly asks for them.

### Governance-maintenance fast path

When the task only edits `AGENTS.md`, `.codex/skills/**`, `.codex/scripts/**`, or their review document:

- Diagnose from the user's observed failure and the current diff only. Do not search the web, reread general best-practice sources, forward-test, create a temporary repository, or expand the methodology unless the user explicitly asks or one concrete unresolved fact blocks the edit.
- Select one root cause, apply one bounded patch, then run exactly one combined validation pass covering only changed artifacts. If it passes, stop immediately and hand off; do not perform reassurance checks.
- Do not update the same rule in every layer by default. Put the decision rule in `AGENTS.md`, domain-only detail in the Skill, and deterministic behavior in the script. Update another layer only when its behavior would otherwise contradict the fix.
- The repository task guard is not required for maintenance of the guard itself or its instruction files. Preserve unrelated dirty files and do not commit/tag unless the user explicitly requests it or this maintenance is already an isolated task-owned change.
- Maximum normal tool sequence: one inspection call, one patch call, one combined validation call. A fourth call requires a failed validation or a concrete blocker, and the reason must be stated before running it.

- A small bug is `quick-fix` by default. Do not promote it to architecture, broad research, or native work merely because more investigation is possible.
- Lane names describe risk and workflow only. They do not impose elapsed-time, changed-file, cycle-count, checkpoint, or replan gates.
- Optimize for shortest elapsed time by removing redundant exploration, repeated commands, speculative scope, and low-value validation. Never gain speed by dropping required behavior, risk-driven verification, rollback, or the requested completion target.
- Under a user-stated time constraint, never repeat a successful or otherwise conclusive check, and never expand research after the available evidence can decide the approved implementation. A retry requires a relevant edit or an inconclusive result; broader research requires one named unresolved fact that can materially change the decision.
- When the user states a time budget or deadline, treat it as the execution target for the current turn and reserve roughly the final quarter for verification, documentation, commit, tag, and handoff. Choose one shortest evidence-backed route, run each expensive build/test at most once unless a relevant edit or an inconclusive result requires a retry, and stop optional research or cleanup before the budget is exhausted. Keep all code, lock, artifact, and task-document changes for one approved unit in one guard session and one commit when possible; only make a second documentation-only closure commit when an unavoidable post-commit ID/tag must be recorded, with no extra build or research pass. This routing rule never lowers correctness, rollback, or completion requirements.
- Classify command failures before asking for approval: a repository file-mode or invocation error is fixed by using the correct in-scope invocation once (for example `bash ./gradlew`), while sandbox/network/permission errors are the cases that warrant an escalation request. Do not spend a second attempt or approval round on the wrong failure class.
- Do not widen declared behavior or paths without explicit user approval. Split genuinely large work into independently useful units while preserving the original completion target.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [webhtv/webhtv](https://github.com/webhtv/webhtv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
