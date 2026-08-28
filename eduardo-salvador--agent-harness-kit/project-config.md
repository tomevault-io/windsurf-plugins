---
trigger: always_on
description: Codex loads this file before work. Claude Code imports it from `CLAUDE.md`. This is the shared, platform-neutral policy map; load details progressively and do not preload the repository.
---

# Agent Harness Kit — operational map

Codex loads this file before work. Claude Code imports it from `CLAUDE.md`. This is the shared, platform-neutral policy map; load details progressively and do not preload the repository.

## First-run gate

Before planning implementation, inspect `harness-state/PROJECT-CONTEXT.md`. Context is initialized only when it contains `schema: harness.project-context/v1` and `status: approved`. If it is absent, draft, stale, or conflicts with current evidence, use the platform's `first-run-discovery` skill and follow [first run](harness/playbooks/first-run.md). Discovery identifies greenfield versus existing-project state, inventories rules and capabilities, records decisions for confirmation, selects `delivery`, `delivery+learning`, `hackathon`, or `hackathon+learning`, obtains approval, and only then creates the initial graph.

If mature harness material exists, do not overwrite `AGENTS.md`, `CLAUDE.md`, `.agents/`, `.claude/`, `.mcp.json`, or another authority. Follow [mature adoption](harness/playbooks/mature-harness-adoption.md): use a namespaced staged installation, classify every material item with provenance and backlinks, validate snapshot freshness, and preserve originals until human semantic review and separate cutover authorization.

## Session-start, resume, and status gate

On the first request in a new context window, any request to continue/resume, or any project-status request, follow [status and resume](harness/playbooks/status-resume.md) before broad inspection:

1. Read `harness-state/PROJECT-CONTEXT.md`.
2. Read the pending-work authority named by approved context/decisions; otherwise use `harness-state/PENDING.md` when present.
3. Read `harness-state/TASK-GRAPH.md`.
4. Only then load the active task, direct graph neighborhood, relevant decisions/rules/capabilities/model routing, and latest handoff/review.

Do not substitute repository-wide scanning, dependency inventory, Git-history traversal, or conversational recall for this order. If an artifact is missing, stale, or contradictory, report that exact condition and enter the applicable discovery/recovery playbook. Broader inspection is allowed only for a concrete gap exposed by these artifacts, a required recovery step, or an explicit user audit request; announce its reason and scope first.

For “my pending items”, “what do you need from me?”, approvals, or decisions, read the pending authority in full and report open human-owned items first, including items outside the graph. State explicitly when there is no recorded human action. Never answer these requests from the graph alone.

## State authority split

- `harness-state/PENDING.md` owns human decisions/actions and the macro project completion overview: product areas or outcomes still missing, such as unfinished backend or authentication. It does not schedule technical tasks.
- `harness-state/TASK-GRAPH.md` owns technical order, dependencies, readiness, leases, dispatch, remediation, and execution state. It does not replace the human/macro pending view.
- Every technical event—dispatch/start, material progress, new dependency, block/unblock, remediation, completion, lease/context change, or next-task readiness—must update `TASK-GRAPH.md` and its transition log in the same operational step, before the user-facing update. Never record technical movement only in `PENDING.md`. Update `PENDING.md` in that step only when a human item or macro project outcome also changed, and backlink its technical source to the new graph revision.
- Status and resume reads both in the required order. “My pending items” is answered from human-owned `PENDING.md` entries first; technical detail is added from the graph only when useful or requested.

## Operational loading order

1. Load the assigned [role](harness/roles/README.md), task brief, pinned context revision, relevant decisions, graph neighborhood, scoped rules, capability manifest, and approved model-routing revision named by the task.
2. Follow the applicable [playbook](harness/playbooks/README.md); use [templates](harness/templates/README.md) for durable state. Files carry state; messages announce changes.
3. Use only approved capabilities. Never assume tools, MCP/connectors, skills, commands, hooks, integrations, authentication, secrets, network, or permissions.
4. Write only within the exclusive ownership lease. The orchestrator alone changes graph topology/status and leases. Implementers never self-accept; reviewers remain independent.
5. Run `python tools/validate.py` before review when Python 3 is available, otherwise follow [the validation contract](docs/VALIDATION.md).
6. Route work by [capability tier](docs/MODEL-ROUTING.md), not prestige: balanced is the normal default; economical requires deterministic low-risk acceptance; frontier is reserved for consequential judgment and escalation triggers. Routing changes no authority.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Eduardo-Salvador/Agent-Harness-Kit](https://github.com/Eduardo-Salvador/Agent-Harness-Kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
