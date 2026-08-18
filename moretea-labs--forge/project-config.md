---
trigger: always_on
description: This repository self-hosts the Forge contract. Retired project-skill and project-initializer staging paths are not supported or cleaned up by current tooling. Claude and Codex should follow the same repo-local workflow surface.
---

# Forge AGENTS.md

This repository self-hosts the Forge contract. Retired project-skill and project-initializer staging paths are not supported or cleaned up by current tooling. Claude and Codex should follow the same repo-local workflow surface.

## Forge execution runtime

Treat ChatGPT as the controller and Forge as its repository execution layer. ChatGPT chooses how to inspect, plan, edit, verify, or delegate. Forge provides deterministic repository tools and does not impose an Agent-first workflow.

- Direct Edit is the default for understood work. One session may accept many patch batches, keep revision history, create savepoints, run checks, roll back selected revisions, and finalize one aggregate localized diff.
- Tasks describe objectives, scope, checks, and acceptance criteria. They do not permanently bind Codex, Claude, or GitHub Copilot. The executor is selected when each Run starts.
- Agents are optional implementation tools for broad exploration, large refactors, or compile/test/fix loops. They receive a high-level implementation contract; ChatGPT still reviews the result and decides what happens next.
- Ordinary local risk levels are metadata, not permission gates. There is no approval queue and no `approve_risk` handshake. Only an explicitly destructive or irreversible operation requires authorization in the same request.
- The Controller UI is an auxiliary configuration/state utility behind ChatGPT: Overview, Work, Automations, Capabilities, Repositories, Settings, and System. It presents durable user-facing state and hides Issue/Task/Run internals unless diagnostics require them.
- Hard runtime boundaries remain for secrets, credentials, Git internals, concurrent write conflicts, out-of-scope writes when a scope is declared, and remote or irreversible side effects.
## Canonical Workflow Files

- `tasks/current.md` for the tracked current-status snapshot derived from workflow artifacts
- `tasks/todos.md` for deferred medium/long-term goals, not active execution checklists
- `plans/prds/` for upper-layer PRDs; `plans/sprints/` for ordered sprint backlogs operated through `scripts/sprint-backlog.sh` (installed implementations live under `.ai/harness/scripts/`; task contracts stay the execution slices)
- `.ai/context/capabilities.json` for the capability registry and longest-prefix context boundaries
- `tasks/workstreams/` for capability long-running workstreams that project durable progress into local contracts
- `tasks/lessons.md` for correction-derived rules
- `docs/researches/` for deep repo knowledge
- `tasks/notes/` for task-local implementation decisions, deviations, tradeoffs, and open questions
- `plans/` for timestamped plans, with `plans/archive/` for history
- `.ai/harness/workflow-contract.json` for the installed workflow contract manifest
- `.ai/harness/policy.json` for the machine-readable workflow contract
- `.ai/context/context-map.json` for progressive context loading
- `docs/architecture/index.md` for umbrella architecture status, drift requests, snapshots, and diagram links
- `docs/reference-configs/agentic-development-flow.md` for Forge task routing and optional host-skill enhancements

## Runtime Architecture Guardrails

- Treat Forge as one local MCP application, one active Runtime, one deployable release, and one in-process lifecycle owner. Gateway, MCP transport, Controller, and Scheduler may be modules; they are not independently deployable generations by default.
- Bounded restart downtime is acceptable. Prefer `stop -> switch complete release -> start -> verify -> full rollback`; do not add ingress, blue/green slots, adoption, or mixed generations without an explicit product requirement.
- Do not respond to an incident by adding a second status authority, daemon, proxy, KeepAlive wrapper, watchdog, recovery owner, or fallback path; extend the canonical Runtime or standalone Recovery owner instead. First identify the violated invariant and remove, merge, or correct the existing cause.
- Readiness is one derived whole-system conclusion. Multiple diagnostic checks and reason codes are allowed, but they must not become independent durable readiness state machines.
- Keep lifecycle, readiness, liveness, capability, authorization, release identity, and diagnostics separate. Do not create composite states such as `status=ready` with `degraded=true`.
- Only one component may perform lifecycle or recovery side effects. Observers and probes are read-only and submit typed requests to that owner.
- Release and rollback scope is the complete compatible Runtime, configuration, entrypoint, and SQLite schema/backup state; component-level rollback is forbidden.
- Any new process, persistent state, enum value, health mode, authority file, or compatibility fallback requires an explicit architecture decision, transition owner, cleanup/removal criterion, and failure-injection tests.
- Follow `docs/reference-configs/runtime-architecture-guardrails.md` for the normative review gates and target topology.

## Operating Rules

- Sync `tasks/` whenever substantive repo changes are made.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moretea-labs/forge](https://github.com/moretea-labs/forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
