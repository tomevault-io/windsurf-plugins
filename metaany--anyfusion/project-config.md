---
trigger: always_on
description: AnyFusion is the public product name. `MetaClaw`, `metaclaw`, and `Metaclaw*`
---

# Repository Guidelines

## Start Here

AnyFusion is the public product name. `MetaClaw`, `metaclaw`, and `Metaclaw*`
remain internal/runtime names and the compatibility CLI alias. Do not rename them
during unrelated work.

Read only what the task needs, in this order:

1. [`CONTEXT.md`](CONTEXT.md) — current contracts, domain vocabulary, runtime
   invariants, and product boundaries.
2. [`docs/current/technical-overview.md`](docs/current/technical-overview.md) —
   full runtime, deployment, configuration, and repository overview.
3. [`docs/adr/README.md`](docs/adr/README.md) — accepted decisions and the
   authority matrix. Read [ADR-0020](docs/adr/0020-core-module-ownership-and-dependency-direction.md)
   before architecture or roadmap work.
4. [`docs/README.md`](docs/README.md) — current docs, plans, operational notes,
   and archives.

Authority order: code and tests, accepted ADRs, [`CONTEXT.md`](CONTEXT.md), current
technical docs, active plans, then archived material. [`docs/archive/`](docs/archive/) is
historical unless a current authority explicitly cites it.

Architecture shortcuts:

- Planner: [ADR-0015](docs/adr/0015-planner-owned-semantics-and-tool-mediated-context.md)
- AgentClass definitions/status: [ADR-0018](docs/adr/0018-supported-routing-contracts-and-unified-executor-definitions.md), [ADR-0017](docs/adr/0017-kernel-executor-status-projection.md)
- Work Graph/publication: [ADR-0021](docs/adr/0021-work-graph-v4-subtask-execution-contract.md), [ADR-0025](docs/adr/0025-single-task-concurrency-and-git-publication.md), [ADR-0026](docs/adr/0026-phase-6-single-task-reliability-closure.md)
- Kernel/recovery: [ADR-0022](docs/adr/0022-unified-kernel-control-plane-and-decision-ledger.md), [ADR-0023](docs/adr/0023-durable-kernel-workflow-recovery-and-availability.md)
- Sandbox/resources: [ADR-0024](docs/adr/0024-resource-partition-sandbox-and-runtime-elevation.md), [runtime security](docs/current/phase-5-runtime-security.md)
- Single-Task boundary: [ADR-0011](docs/adr/0011-single-active-task-admission-gate.md), [future roadmap](docs/plans/future-multi-task-scheduling-roadmap.md)

## Repository Map

MetaClaw is a Node 22.19+ TypeScript ESM CLI/TUI. `src/index.ts` is the composition
root. Detailed ownership and dependency rules live in
[ADR-0020](docs/adr/0020-core-module-ownership-and-dependency-direction.md).

| Area | Start here |
| --- | --- |
| Planning and AnyFusion-Pi session | [`src/planning/`](src/planning/) |
| Pure policy and graph rules | [`src/kernel/`](src/kernel/), [`src/work-graph/`](src/work-graph/) |
| Application Shell | [`src/session/`](src/session/) |
| Attempts, recovery, sandbox, Git publication | [`src/execution/`](src/execution/), [`src/executor/`](src/executor/), [`src/resource/`](src/resource/) |
| Durable facts | [`src/storage/`](src/storage/) |
| Task and explicit memory | [`src/task/`](src/task/), [`src/memory/`](src/memory/) |
| CLI, commands, native TUI bridge, and standby Ink UI | [`src/cli/`](src/cli/), [`src/commands/`](src/commands/), [`src/tui-bridge/`](src/tui-bridge/), [`src/tui/`](src/tui/) |
| Gateway, Feishu, notifications, delivery | [`src/gateway/`](src/gateway/), [`src/integrations/`](src/integrations/), [`src/notifications/`](src/notifications/), [`src/delivery/`](src/delivery/) |
| Supporting domains | [`src/guidance/`](src/guidance/), [`src/learning/`](src/learning/), [`src/intent/`](src/intent/), [`src/core/`](src/core/) |

Main entry points:

- [`anyfusion`](anyfusion) — native Linux server launcher and worktree smoke entry.
- [`src/index.ts`](src/index.ts) — composition and mode selection.
- [`src/session/metaclaw-session.ts`](src/session/metaclaw-session.ts) — Application Shell.
- [`src/planning/anyfusion-planning-agent.ts`](src/planning/anyfusion-planning-agent.ts) and
  [`src/planning/planner-process-runner.ts`](src/planning/planner-process-runner.ts) — Planner boundary.
- [`src/kernel/control-kernel.ts`](src/kernel/control-kernel.ts) and
  [`src/kernel/kernel-workflow.ts`](src/kernel/kernel-workflow.ts) — policy and
  durable control seam.
- [`src/execution/kernel-execution-runtime.ts`](src/execution/kernel-execution-runtime.ts) and
  [`src/execution/subtask-attempt-runner.ts`](src/execution/subtask-attempt-runner.ts) — execution chain.
- [`src/tui-bridge/planner-tui-bridge.ts`](src/tui-bridge/planner-tui-bridge.ts) and
  [`src/tui-bridge/planner-tui-process.ts`](src/tui-bridge/planner-tui-process.ts) — default native Planner TUI adapter.
- [`src/tui/app.tsx`](src/tui/app.tsx) — preserved standby Ink UI; it is not the default local surface.
- [`src/gateway/server.ts`](src/gateway/server.ts) and
  [`src/gateway/feishu-runtime.ts`](src/gateway/feishu-runtime.ts) — remote surfaces.

Tests mirror source domains under [`tests/`](tests/). Scenarios and fixtures are in
[`examples/`](examples/); Docker and smoke orchestration are in [`docker/`](docker/) and
[`scripts/`](scripts/).

## Working Rules

- Preserve ADR-0020's ownership and dependency direction. Detailed runtime rules
  belong in `CONTEXT.md`, not this file.
- The sibling AnyFusion-Pi fork is the default local Planner surface. Its Task panel and
  bridge are presentation/Application-Shell adapters only: they may project state and hand a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MetaAny/AnyFusion](https://github.com/MetaAny/AnyFusion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
