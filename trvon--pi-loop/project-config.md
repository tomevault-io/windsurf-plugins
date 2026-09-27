---
trigger: always_on
description: pi-loop is a Pi extension for scheduled/event-driven re-wakes, self-paced goals, task-driven workflows, bounded subagent orchestration, native fallback tasks, and background process monitoring.
---

# pi-loop development guide

## Purpose

pi-loop is a Pi extension for scheduled/event-driven re-wakes, self-paced goals, task-driven workflows, bounded subagent orchestration, native fallback tasks, and background process monitoring.

Stack: strict TypeScript 7, ES2022, TypeBox, Vitest, and Biome.

## Authority boundaries

Do not blur these domains:

- `LoopStore` exclusively owns loops, workflow definitions/revisions/executions/history/leases, and finite subagent orchestration batches/dispatches.
- `TaskStore` or external `pi-tasks` owns standalone tasks only.
- `/tasks`, task RPC, `TaskClaim`, and `TaskUpdate` never control workflow work.
- `MonitorManager` owns process-local monitor state; monitor recovery across Pi death is not implemented.
- `pi-subagents` owns worker execution and global concurrency. pi-loop owns only session-scoped orchestration intent, bounded evidence, local capacity, and recovery decisions.
- Pending notifications are memory-only; persisted controllers recover on resume, not while Pi is absent.

A feature that requires a cross-store workflow/task transaction violates the architecture.

## Source map

```text
src/index.ts                 extension registration and runtime wiring
src/api.ts                   supported @trevonistrevon/pi-loop/api surface
src/types.ts                 loop, workflow, revision, monitor contracts
src/store.ts                 LoopStore workflow/orchestration atomic mutations
src/workflow-admission.ts    provider-neutral blocker transition admission
src/task-store.ts            standalone native task persistence
src/*-reducer.ts             pure state transitions
src/coordinator.ts           reducer/effect coordination
src/scheduler.ts             cron scheduling and fire times
src/trigger-system.ts        cron/event/hybrid activation
src/monitor-manager.ts       child processes and bounded output
src/runtime/                 session, notification, backlog, orchestration, task-provider, monitor wiring
src/tools/                   model-facing tool definitions
src/commands/                /loop and /tasks
src/rpc/                     vendored cross-extension RPC
src/ui/                      status and tool rendering
test/                        unit, integration, property, and live harnesses
```

## Workflow contract

A workflow is one dynamic `LoopEntry` with a version-1 named-state definition.

- State `task` data materializes as `WorkflowExecutionRecord` in LoopStore.
- The creator owns the initial execution lease.
- Every destination/retry execution starts unowned and requires `WorkflowClaim`.
- `WorkflowTransition` validates the live owner, settles source work, records evidence, advances state, and creates destination work in one locked write.
- Paused terminal outcomes require a typed blocker claim; trusted providers run outside the LoopStore lock, then the transition uses exact state/revision/execution CAS.
- Machine observations never grant user authority. Rejected, stale, or contradicted claims are state-preserving; restart recovery is explicit resubmission, not a persisted proposal.
- `WorkflowRevise` applies typed definition changes with definition/state/sequence CAS and immutable prior-definition history; it never touches TaskStore.
- Active instructions are never mutated in place. `reissue_state` cancels the prior execution into history, creates fresh current work under the still-valid lease, and wakes the replacement after the locked write; through an administrative pause it parks the replacement instead of waking it. Current outgoing edges and future state content remain directly revisable.
- Transition CAS includes definition revision so transition/revision races fail closed in either order.
- Terminal completed workflows are deleted; terminal paused workflows remain inspectable with bounded admission and pause provenance.

## Subagent orchestration contract

An orchestration is one finite, session-file-backed `LoopEntry` batch.

- It accepts explicit independent work only; it never discovers TaskStore work or workflow executions.
- Creation requires protocol-v2 `pi-subagents` and rejects memory/project/custom/off storage.
- Every dispatch is persisted before spawn and fenced by controller revision, owner runtime/generation, work ID, dispatch ID, attempt, and upstream agent ID.
- `spawning`, `queued`, and `running` consume local capacity; pi-subagents retains its global queue.
- Lifecycle evidence is bounded and persisted before `subagents:rpc:consume`.
- Proved failures may retry within the item budget. Ambiguous timeout/recovery never retries automatically.
- The existing session heartbeat reconciles orchestration; do not add another timer or scheduler.
- Session teardown invalidates callbacks before best-effort stop and state reconciliation.
- Project scope, dependency graphs, dynamic work addition, and exactly-once dispatch remain unsupported.

See `docs/REFERENCE.md` for the public contract.

## Persistence and lifecycle

Reducer-backed stores use:

- initialized PID/UUID owner claims published by exclusive hard link, with sole-claim admission and stale-owner detection;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trvon/pi-loop](https://github.com/trvon/pi-loop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
