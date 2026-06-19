---
trigger: always_on
description: >-
---


# Agentic Services Orchestrator Skill

## Purpose

Coordinate workflow state, routing, sequencing, dependency tracking, handoff contracts, missing-info handling, approval gates, validation, recovery, and plugin-weaving across configurable workflow domains. The CompleteTech LLC agentic services lifecycle is the built-in default adapter, not the hardcoded core. Specialist skills own their artifacts.

For the complete architecture, per-skill responsibility matrix, handoff schema, plugin-weaving model, deduplication guidance, and example multi-skill workflows, load `references/orchestration-architecture.md`. For schema-driven routing, load `references/workflow-definition-schema.yaml` and the default adapter `references/completetech-services-workflow.yaml`.

## System Boundary

This skill owns workflow definition loading, routing policy, project state, event logging, validation, recovery selection, and specialist handoff coordination. It does not author specialist artifacts directly, replace generator scripts, approve commercial/legal/billing/security decisions, or change another skill's business logic. Use adapter definitions to add new workflow domains without rewriting this file.

## Runtime Permissions

This skill is a local workflow-orchestration and document-rendering workflow. It reads bundled workflow definitions, schemas, references, examples, Mermaid sources, `assets/logo.png`, and user-provided project-state/workflow facts. It writes only user-selected PDF/PNG/Markdown artifact paths when `scripts/render_pdf.py` is explicitly run.

It does not send emails, call external workflow systems, call Mautic or other CRMs, contact certificate services, require credential access, create persistence, escalate privileges, perform destructive file operations, or run background services.

## Network Boundary

This skill is local-only. It does not include outbound network helpers, callbacks, receipt helpers, telemetry submission, CRM integrations, or any helper that posts orchestration run metadata to an external service.

## Universal Workflow Model

The orchestrator is based on generic workflow primitives:

- `workflow_type`: the domain adapter key, such as `completetech_services`, `hiring_pipeline`, or `software_release`.
- `stage`: named lifecycle position in a workflow definition.
- `track`: parallel workstream with stage, status, owner, dependencies, blockers, and output limits.
- `artifact`: versioned work product with source artifacts and approval impact.
- `actor` / `owner`: person, role, specialist skill, plugin, or external approver responsible for an action.
- `decision`: recorded choice with rationale, evidence, and affected tracks/artifacts.
- `gate`: approval, risk, quality, policy, compliance, or authority checkpoint.
- `dependency`: fact, artifact, approval, event, or external condition needed before action.
- `event`: append-only state change in the workflow event log.
- `state_transition`: movement between stages/tracks/statuses allowed by the workflow definition.
- `recovery_action`: smallest useful safe action when work is blocked or underspecified.

Default workflow adapter: `references/completetech-services-workflow.yaml`.

Default CompleteTech lifecycle: Discovery -> Proposal -> Contract -> Delivery -> Customer Success.

Real engagements may move forward, loop backward, skip a stage, reopen an approval, split into parallel tracks, stall, or branch into change-order work. Treat the lifecycle as a state map, not a strict pipeline.

Supporting outputs: Invoice, Certificate, Case Study, Email, Envelope.

Overlay/gate: Security Review. Security is not the default gate for every workflow; use approval/risk triage first. Route ordinary commercial, legal, billing, external-send, public-proof, and client-authority approvals to the appropriate owner or specialist. Invoke security review only when sensitive data, permissions, credentials, new tools/integrations, production launch, external tool actions, public proof confidentiality, incident response, or material security/compliance risk is involved.

## Routing Logic

1. Load the applicable workflow definition from `project_state.workflow_type`; use `completetech_services` when unspecified.
2. Classify intent: create, revise, continue, route, package, send, review, approve, escalate, recover, archive, validate, or start a new workstream.
3. Identify current state from workflow definition, explicit user context, active tracks, existing artifacts, prior handoff notes, approval history, event log, blockers, due dates, and known conflicts.
4. Decide whether the request is forward progress, backward rework, a skipped-stage exception, a reopened approval, a continuation, a revision, an escalation, a packaging task, or a parallel workstream.
5. Check `allowed_transitions`, `routing_rules`, `required_fields`, `gates`, and `specialist_owners` from the workflow definition before routing.
6. Check artifact versions before creating anything new. Revise, supersede, fork, archive, or reference the existing artifact when that is the cleaner state transition.
7. Route by requested outcome, current stage/track, artifact versions, missing facts, conflicts, dependencies, approvals, risk/approval triage, urgency, owner, allowed transitions, and domain adapter rules.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CompleteTech-LLC/agentic-services-orchestrator-skill](https://github.com/CompleteTech-LLC/agentic-services-orchestrator-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
