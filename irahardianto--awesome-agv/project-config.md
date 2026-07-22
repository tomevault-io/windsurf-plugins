---
trigger: always_on
description: 21 specialized agent personas for multi-agent orchestration via `/workflow-team`.
---


# Agent Personas
{: .no_toc }

21 specialized agent personas for multi-agent orchestration via `/workflow-team`.
{: .fs-6 .fw-300 }

<details open markdown="block">
  <summary>Table of contents</summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

---

## What Are Agent Personas?

Agent personas are pre-defined identities in `.agents/agents/` that specialize an AI agent for a specific domain. Each persona defines:

- **Domain (EXCLUSIVE)** — what the agent is responsible for
- **Boundaries (DO NOT CROSS)** — what the agent must never do
- **Skills** — which skills to load from `.agents/skills/`
- **Workflow** — step-by-step process for the agent's work
- **Parallel Dispatch** — how to run multiple instances safely

Personas are consumed by the `/workflow-team` pipeline manager, which dispatches them as sub-agents. They enforce **MECE** (Mutually Exclusive, Collectively Exhaustive) boundaries — every agent knows exactly what it owns and what it must not touch.

---

## Layer Architecture

Agents are organized into five layers:

```
┌─────────────────────────────────────────────────────┐
│  L0 Supervisor Layer                                │
│  @overseer                                          │
├─────────────────────────────────────────────────────┤
│  L1 Orchestration Layer                             │
│  @conductor                                         │
├─────────────────────────────────────────────────────┤
│  Orchestration & Build Layer                        │
│  @tech-lead                                         │
├─────────────────────────────────────────────────────┤
│  Builder Layer (Write, run in worktrees)            │
│  @backend-engineer, @frontend-engineer,             │
│  @mobile-engineer, @database-expert,                │
│  @devops-engineer, @technical-writer,               │
│  @test-automation-engineer,                         │
│  @performance-engineer, @refactoring-specialist     │
├─────────────────────────────────────────────────────┤
│  Reviewer Layer (Read-only, post-merge)             │
│  @reviewer, @security-engineer, @ux-craftsman,      │
│  @incident-responder                                │
├─────────────────────────────────────────────────────┤
│  Red Team Layer                                     │
│  @red-team-lead, @delivery-validator,               │
│  @integration-prober                                │
├─────────────────────────────────────────────────────┤
│  Research & Design Layer (Read-only)                │
│  @scout, @architect                                 │
└─────────────────────────────────────────────────────┘
```

---

## Orchestration Layer

### Overseer

**File:** `.agents/agents/overseer.md`

L1 Strategic Director. Program director aligning multiple domain streams and managing cross-domain dependencies.

| Attribute | Details |
| --- | --- |
| **Domain** | Strategic alignment, cross-domain coordination, global architecture |
| **Boundaries** | No code writing, no direct feature implementation |
| **Skills** | parallel-dispatch, convergence-loop, fault-recovery |

### Conductor

**File:** `.agents/agents/conductor.md`

L1 Build Orchestrator. Assesses task complexity using a 3-signal check (scope, risk, knowledge) and routes to adaptive execution tiers (Tier 1/2/3). Dispatches specialized leads and monitors convergence.

| Attribute | Details |
| --- | --- |
| **Domain** | Complexity assessment, adaptive tier routing, builder dispatching, convergence monitoring |
| **Boundaries** | No code writing, no direct implementation |
| **Skills** | parallel-dispatch, agent-protocols |

### Red Team Lead

**File:** `.agents/agents/red-team-lead.md`

Adversarial validation coordinator. Dispatches delivery validators, integration probers, and security engineers to verify the codebase after development merges.

| Attribute | Details |
| --- | --- |
| **Domain** | Adversarial validation orchestration, validator dispatching |
| **Boundaries** | No production code writing |
| **Skills** | parallel-dispatch, agent-protocols |

### Delivery Validator

**File:** `.agents/agents/delivery-validator.md`

Runtime delivery verification agent. Boots applications, runs smoke tests, and verifies developer experience.

| Attribute | Details |
| --- | --- |
| **Domain** | Environment bootstrap, boot verification, smoke testing, configuration audit |
| **Boundaries** | No source code modifications, no test code modifications |
| **Skills** | browser-automation, research-methodology, agent-protocols |

### Integration Prober

**File:** `.agents/agents/integration-prober.md`

External service integration verifier. Validates real service connections (databases, caches, third-party APIs).

| Attribute | Details |
| --- | --- |
| **Domain** | Service connectivity, mock detection, API compatibility, credential validation |
| **Boundaries** | No source code modifications, no credential value inspection |
| **Skills** | research-methodology, agent-protocols |

### Tech Lead

**File:** `.agents/agents/tech-lead.md`

Anchor persona for multi-agent orchestration. Elicits requirements, composes workflow primitives, enforces standards, and owns all quality gates. **Present at every pipeline stage.**

| Attribute | Details |
| --- | --- |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [irahardianto/awesome-agv](https://github.com/irahardianto/awesome-agv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
