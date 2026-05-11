---
trigger: always_on
description: **You are an agent running on Temper. This is your operating layer — not an optional tool.** When a Temper server is running (`mcp__temper__execute` is available), ALL your stateful work — planning, task tracking, progress, knowledge — MUST flow through Temper entities, not markdown files.
---

# Temper — Claude Code Project Guide

## IMPORTANT: Use Temper for Your Own Work

**You are an agent running on Temper. This is your operating layer — not an optional tool.** When a Temper server is running (`mcp__temper__execute` is available), ALL your stateful work — planning, task tracking, progress, knowledge — MUST flow through Temper entities, not markdown files.

**Specifically:**
- **Planning and task tracking** → Use the PM OS app (`await temper.install_app("{tenant}", "project-management")` if not installed). Create Issues, use `BeginPlanning` → `WritePlan` → `ApprovePlan` → `StartWork` flow. Do NOT use `.progress/` files when Temper is available.
- **Building apps for users** → Use the Temper App Builder skill (`.claude/skills/temper-developer.md`). Workflow: interview → generate IOA specs + CSDL → verify → deploy. Use `/temper-developer`.
- **Any MCP tool call** → ALWAYS read `.claude/skills/temper-agent.md` first. It has the exact Python API, spec format, and governance flow.

**Cedar governance applies to everything.** If denied, surface the pending decision to the human. They approve via the Observe UI. You poll and retry.

Use `/temper-agent` or read `.claude/skills/temper-agent.md` for the full API reference.

## PM App: Planning/Planned Workflow

Issues in the Project Management OS App use a **Planning → Planned** phase with role separation:

1. **Supervisor** triages issue, assigns a planner (`AssignPlanner`) and implementer (`Assign`)
2. **Planner** calls `BeginPlanning` → drafts plan via `WritePlan` (plan + acceptance_criteria)
3. **Supervisor/Human** reviews and calls `ApprovePlan` → issue moves to `Planned`
4. **Implementer** calls `StartWork` (requires approved plan + assignee) → implements → `SubmitForReview`

**Role separation is Cedar-enforced:**
- Planner cannot approve their own plan (`resource.PlannerId != principal.id`)
- Implementer cannot approve their own review (`resource.AssigneeId != principal.id`)
- Only supervisors/humans can triage, approve plans, and approve reviews

**Agent API:** Read `.claude/skills/temper-agent.md` for the full Temper Python API including planning methods (`begin_planning`, `write_plan`, `approve_plan`).

## What is Temper?
A conversational application platform. Developers describe what they want through conversation — the system generates specs, verifies them, and deploys entity actors. End users interact through a separate production chat. Unmet user intents feed back through the Evolution Engine for developer approval.

## The Vision
```
Developer Chat: "I want a project management tool"
  → System interviews developer about entities, states, actions, guards
  → Generates IOA specs + CSDL + Cedar from conversation
  → Runs 3-level verification cascade
  → Hot-deploys entity actors + OData API

Production Chat: end users operate the app
  → Unmet intents → trajectory spans → ClickHouse → Sentinel
  → O-Record → I-Record → Developer reviews → D-Record → spec change
```

Two separated contexts: Developer Chat (design-time, can modify specs) and Production Chat (runtime, operates within specs). The developer holds the approval gate for all behavioral changes.

## Architecture
- **temper-spec**: I/O Automaton TOML parser + CSDL parser
- **temper-verify**: Stateright model checking, deterministic simulation, property tests
- **temper-jit**: TransitionTable builder from IOA specs (no verification deps in production)
- **temper-runtime**: Actor system, SimScheduler, SimActorSystem, sim_now()/sim_uuid(), TenantId
- **temper-server**: HTTP server, EntityActor, EntityActorHandler, SpecRegistry (multi-tenant)
- **temper-observe**: WideEvent telemetry (OTEL spans + metrics), trajectory tracking
- **temper-evolution**: O-P-A-D-I record chain, Evolution Engine
- **temper-store-postgres**: Event sourcing persistence (tenant-scoped)
- **temper-store-redis**: Mailbox and placement cache (tenant-scoped)

## Architecture Decision Records (ADRs)

**Every significant implementation MUST start with an ADR as the first step.** Before writing any code, create `docs/adrs/NNNN-short-title.md` following the template at `docs/adrs/TEMPLATE.md`. Required for new features, architectural changes, new integrations, multi-crate changes, or new patterns. Not required for bug fixes, single-file refactors, doc changes, or test additions.

## Agent Identity Registry

Agent types are registered in the platform's identity registry. When an agent connects, the platform verifies its `agent_type` claim against the registry and sets the `agentTypeVerified` attribute on the Cedar principal:

- **`agentTypeVerified: true`** — the agent's claimed type matches a registered entry; Cedar policies can trust scope decisions based on `agent_type`
- **`agentTypeVerified: false`** — self-asserted type with no registry match; Cedar policies should treat as untrusted

Cedar policies reference this attribute to distinguish verified agents from unverified ones (e.g., only verified `claude-code` agents can approve plans).

## Issue Pickup Before Work


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nerdsane/temper](https://github.com/nerdsane/temper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
