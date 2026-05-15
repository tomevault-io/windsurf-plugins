---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

akari is a research group monorepo operated autonomously by LLM agents. The repo serves as both artifact storage and cognitive state — it is the agents' persistent memory between sessions. See [docs/design.md](docs/design.md) for rationale.

## Work cycle

The conversation is ephemeral; the repo is permanent. Record as you go, not at the end.

- **Finding → file, immediately.** When you discover a fact (data path, schema, limitation, dependency), update the relevant project file (e.g., `existing-data.md`, `datasets.md`) in the same turn. Do not defer recording to later.
- **Decision → log or decision record.** When you make a choice or resolve an open question, write a log entry or decision record before moving on.
- **Plan → plans/ directory.** When you produce a non-trivial plan, write it to `plans/<name>.md` in the project directory. Plans in conversation history are lost.
- **Session summary → log entry.** Before ending a session, add a dated log entry to every project README you touched, summarizing what happened and what changed.
- **Open questions → README.** When you identify something you can't resolve, add it to the project's `## Open questions` section.

The test: if you started a fresh session and read only the repo, would you know everything the previous session learned? If not, something is missing.

### Knowledge output

akari is a research institute, not a task runner. Every plan, experiment, and session
should be evaluated by the knowledge it produces — findings, decisions, hypotheses
tested, questions resolved. Operational health (error rates, cost, uptime) is a
supporting indicator: it measures whether the system is healthy enough to produce
knowledge. The fundamental efficiency metric is findings per dollar.

Before any implementation plan, ask: "What knowledge does this produce?" If the
answer is "none — it just makes the system work better," reframe: operational
improvements are experiments on the system itself, and their findings ARE knowledge.

**Inline logging checklist** (see `decisions/0004-inline-logging.md`):

1. Discovery of a non-obvious fact → write to project file **in the same turn**, before proceeding.
2. Config/env change → log entry with before/after and rationale, immediately.
3. Successful verification → log the exact command and output (not just "tested successfully").
4. Log incrementally throughout the session. A single end-of-session summary is a fallback, not the primary mechanism.
5. **Findings provenance:** Every numerical claim in an EXPERIMENT.md Findings section must include either (a) the script + data file that produces it, or (b) inline arithmetic from referenced data (e.g., "96/242 = 39.7%"). Claims without provenance are unverifiable and should be treated as suspect by downstream sessions.

## Autonomous execution

This repo runs autonomous agent sessions via [`infra/scheduler/`](infra/scheduler/README.md).
Each session follows the SOP at [docs/sops/autonomous-work-cycle.md](docs/sops/autonomous-work-cycle.md).
See [decisions/0005-autonomous-execution.md](decisions/0005-autonomous-execution.md) for rationale.

For governance (project priority, task selection, goal-directed planning, approval gates), see [docs/conventions/governance.md](docs/conventions/governance.md).

For session discipline (fire-and-forget, sleep limits, incremental commits, analysis throttling), see [docs/conventions/session-discipline.md](docs/conventions/session-discipline.md).

For enforcement layers (L0 code-enforced and L2 convention-only tables), see [docs/conventions/enforcement-layers.md](docs/conventions/enforcement-layers.md).

For task lifecycle tags, fleet-first task creation, and task decomposition, see [docs/conventions/task-lifecycle.md](docs/conventions/task-lifecycle.md).

Tasks are selected from project `TASKS.md` files. Priority order:
1. Tasks in the project recommended by /orient (respecting project priority)
2. Unblocked tasks (no `[blocked-by: ...]` tag)
3. When project budget is >90% consumed or exhausted, prefer `[zero-resource]` tasks
4. Tasks with concrete "Done when" conditions
5. Routine tasks before resource-intensive tasks
6. Tasks with explicit `Priority: high` before `Priority: medium` before untagged

### Approval gates

Autonomous sessions MUST NOT proceed with:
- **Resource decisions**: Requests to increase `budget.yaml` limits or extend deadlines
- **Governance changes**: Changes to approval workflow, budget rules, or other governance mechanisms in CLAUDE.md. Convention clarifications, gotcha additions, and skill improvements may be applied directly — they are verifiable and do not change governance. When in doubt, the test is: "Does this change what requires approval or how resources are allocated?" If yes, it's governance; write to APPROVAL_QUEUE.md.
- **Tool access**: Requests for tools, APIs, or model access not currently configured (see [decisions/0024](decisions/0024-tool-access-approval.md))

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [victoriacity/openakari](https://github.com/victoriacity/openakari) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
