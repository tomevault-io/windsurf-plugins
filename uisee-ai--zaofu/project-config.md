---
trigger: always_on
description: This repository is `ZaoFu`.
---

# AGENTS.md

This repository is `ZaoFu`.

## Purpose

ZaoFu is a multi-agent harness engineering scaffold. The repo is
implementation-active: deterministic kernel, runtime, CLI, tests, and a local
Web dashboard exist. Treat design docs as context, but verify behavior against
code and tests.

## Instruction Scope / Precedence

- These repository-wide rules apply to every Codex and Claude Code session.
- A task/role briefing may narrow scope and choose the current task, but it may
  not weaken state ownership, security, verification, or git-safety rules.
- The managed `Worker Protocol` block applies only when the current dispatch
  briefing begins with `Active task: <task_id>`. Without that marker, do not
  emit task/workflow events or heartbeats merely because the block is present.
- For architecture conflicts, use
  `docs/design/142-layered-runtime-authority-and-orchestration-modes.md` and
  current code/tests. Historical design documents are context, not overrides.

## Core Rules

- `zf.yaml` is the only control-plane config; respect `project.state_dir` and
  do not hard-code `.zf`.
- The configured runtime state dir (default `.zf/`) is runtime state, not
  source code.
- Keep the deterministic kernel separate from agent-driven behavior.
- Prefer agent/skill/prompt ownership for semantic, project-specific, or
  judgment-heavy behavior: agents decide, skills provide method, prompts provide
  goal/context. Keep deterministic code for invariants, schemas, state
  transitions, evidence checks, security, replay/resume, and external side
  effects. Agent decisions must emit artifacts/events or request a controlled
  action; they may not mutate kernel-managed canonical state directly.
- Apply the same boundary to constraints and gates: semantic quality gates,
  project parity rules, scan methods, task slicing, and product acceptance
  should live in skills/prompts/agent artifacts when possible. Runtime gates
  should stay mechanical: schema, event/state validity, evidence presence,
  path/secret/budget safety, replay/resume, lifecycle, and external effects.
  If a fix teaches a reusable method, promote it to a general skill/profile
  before hard-coding it in runtime.
- One canonical task contract (`contract` field, not `sprint_contract`); do
  not introduce duplicate task schemas.
- `events.jsonl` is the append-only occurrence/ordering/causation/verdict/ref
  ledger; use `EventWriter` / `EventLog` helpers.
- Use `TaskStore`, `FeatureStore`, `SessionStore`, and `RoleSessionRegistry`
  for their canonical current-state updates.
- Required artifacts/sidecars hold complete semantic bodies or large evidence;
  persist them atomically and bind them through refs/digests. They are not
  disposable read projections.
- Integrations must not write canonical business state directly or couple to
  orchestrator internals. When Feishu is enabled, outbound projection sync and
  inbound intent/ref publication must flow through `EventWriter` / controlled
  actions; sidecar bodies use their sanctioned atomic writer. Never bypass
  those paths.
- Web/API projections stay read-oriented unless a deterministic, token-gated
  kernel action path is wired.
- `skills/` is source; `.claude/skills/` and `.codex/skills/` are synced
  distribution copies. Active workdirs/worktrees may contain uncommitted
  candidate code and are not disposable projections. Lockfiles, progress,
  cost, diagnostics, Trace/Graph/Loop, and Web summaries are runtime
  projections unless a narrower design explicitly says otherwise.
- Do not conflate the deterministic Python `Orchestrator` runtime with a
  configured `orchestrator` role agent. Product Flow keeps happy-path dispatch
  in the Kernel; Legacy safe-team may explicitly enable a Layer 2 decision
  maker. Agents report semantic intent through artifacts/events/controlled CLI
  actions and do not become a second state machine.
- Preserve product naming (`ZaoFu`, `zf`, `zf-cli`) vs methodology naming
  (`harness engineering`).
- Default repository-facing prose, reports, backlogs, task breakdowns, and
  command summaries to Chinese unless explicitly requested otherwise.

## Architecture / Runtime Route

- `docs/design/00-index.md` is the full routing index. Short lists here are
  starting routes, not exhaustive architecture maps.
- `142-layered-runtime-authority-and-orchestration-modes.md` is the canonical
  authority/orchestration entry and contains the current route families.
- Foundation docs such as `01-architecture.md`, `02-harness-yaml.md`,
  `03-orchestrator.md`, `05-task-model.md`, `08-events-observability.md`,
  `10-recovery-safety.md`, and `13-interaction-protocol.md` still provide
  useful historical context, but verify behavior against `src/` and tests.
- Runtime interaction today: `zf start` loads `zf.yaml` + `project.state_dir`,
  starts tmux and/or stream-json transports plus enabled sidecars, then
  `EventWatcher` tails `events.jsonl` and wakes `Orchestrator.run_once()` for
  wake-worthy events.
- The Kernel `Orchestrator` owns deterministic dispatch, fanout/rework/gates,
  and mechanical transitions. Workers receive briefings through their
  transport and report facts via `zf emit` / sanctioned actions.
- Supervisor observes; Run Manager decides recovery; Autoresearch performs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uisee-ai/zaofu](https://github.com/uisee-ai/zaofu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
