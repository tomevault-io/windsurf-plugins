---
trigger: always_on
description: >
---


---
name: ai-engineering-runtime
version: 10.0.0
description: >
  Use when building, architecting, deploying, auditing, or scaling software systems.
  Triggers for: production APIs, SaaS products, AI/LLM systems, distributed infrastructure,
  security audits, reliability reviews, architecture critiques, and deployment pipelines.
---

# AI Engineering Runtime — Adaptive Orchestration Adapter

This skill does NOT statically inject the full V6 runtime.
It classifies the task, selects a runtime tier, and activates only the modules that add value.

---

## STEP 1 — EXECUTION STATE MACHINE

All engineering tasks pass through these states in order. Skip states only when tier permits.

```
[1]  TASK_CLASSIFICATION   → detect complexity, domain, environment, performance profile
[2]  SHARED_CONTEXT_LOCK   → establish cross-module assumptions + recall memory (see STEP 2 below)
[3]  CONSTRAINT_DETECTION  → surface blockers, risks, ambiguities
[4]  MODE_SELECTION        → select tier + mode (see below)
[5]  EXPERT_ACTIVATION     → activate only required specialists
[6]  ARCHITECTURE_PASS     → only if T2+
[7]  IMPLEMENTATION_PASS   → always — this is the core deliverable
[7b] TOOL_ROUTING          → detect stack, load tool sequence (T2+)
[8]  VALIDATION_PASS       → EXECUTABLE: lint→test→tdd_check→security→schema→ai_eval→deploy→verification
                             TWO-STAGE REVIEW: (1) spec compliance, (2) code quality
[9]  DEPLOYMENT_PASS       → only if T2+
[10] AUDIT_PASS            → only if T4 or AUDIT mode
[11] FINAL_OUTPUT          → scaled to tier
```

State machine reduces drift, hallucination, and inconsistent outputs.

---

## STEP 2 — SHARED CONTEXT CONTRACT

Before any module activates, establish the shared runtime context. All modules read from this — none may make conflicting assumptions.

```
SHARED CONTEXT (resolve at step [2], carry through all steps)

product_stage:       [prototype | mvp | production | enterprise]
environment:         [local | startup | saas | enterprise | ai-native |
                      edge | mobile-backend | high-scale | cost-constrained |
                      regulated | air-gapped | multi-tenant |
                      serverless | hybrid-cloud | gpu-inference | streaming |
                      memory-constrained | cpu-constrained | privacy-consumer |
                      low-trust-integration]
performance_profile: [latency-sensitive | throughput-heavy | batch |
                      interactive | background | standard |
                      gpu-bound | memory-bound | io-bound |
                      streaming-realtime | burst-traffic | event-fanout]
infra_constraints:   [e.g. no Kubernetes, managed DB only, single region]
compliance_scope:    [none | SOC2 | HIPAA | GDPR | PCI | internal-only]
scaling_assumption:  [single-user | <10k | 10k-1M | >1M]
cost_sensitivity:    [low | medium | high / FinOps-first]
observability_req:   [minimal | standard | full]
```

Infer from context where not stated. State assumptions explicitly if inferred.
Conflict between modules → escalate to user before proceeding.

For contract enforcement rules (immutable keys, mutation scope, conflict resolution): load `modules/contracts.md`

For full environment and performance profiles, load: `modules/environments.md`

---

## STEP 3 — COMPLEXITY TIERING

Classify the request before doing anything else.

| Tier | Type | Examples |
|------|------|----------|
| T0 | Simple utility | regex, script, bug fix, type conversion |
| T1 | Standard feature | isolated function, small module, CLI tool |
| T2 | Production feature | API endpoint, SaaS feature, deployed service |
| T3 | Platform/AI system | multi-service, agent, RAG, LLM orchestration |
| T4 | Enterprise/critical | distributed system, high-availability, compliance |

**Tier determines everything downstream.** When in doubt, classify conservatively (lower tier).

---

## STEP 4 — RUNTIME MODE SELECTION

Select exactly one mode based on task domain. Modes determine which reference modules to load.

| Mode | Activates For | Load Module |
|------|--------------|-------------|
| FAST | T0–T1: utilities, fixes, scripts | *(no module — inline execution)* |
| PRODUCT | T2: SaaS, APIs, apps | `modules/product.md` |
| AI SYSTEMS | T2–T3: agents, RAG, LLM, MCP | `modules/ai-systems.md` |
| INFRA | T2–T4: CI/CD, Docker, K8s, cloud | `modules/infra.md` |
| AUDIT | Any tier: security/reliability review | `modules/audit.md` |

Only load the module that matches. Do not load all modules.

**Composite tasks (span multiple modes):** select the PRIMARY mode based on what the user
is actually asking for. Load the secondary module only if it materially changes the output.

| Task | Primary | Secondary |
|------|---------|-----------|
| "Build a SaaS API with Docker deployment" | PRODUCT | INFRA |
| "Deploy a RAG pipeline to Kubernetes" | AI SYSTEMS | INFRA |
| "Security audit of our CI/CD pipeline" | AUDIT | INFRA |
| "Build + deploy a multi-tenant API" | PRODUCT | (INFRA sections inline) |

When in doubt: load the primary module fully. Pull specific patterns from secondary inline.

Load `modules/environments.md` alongside any T2+ module when environment profile or performance profile materially affects the architecture.

---

## STEP 5 — RUNTIME BUDGETING

Before generating output, budget the response:

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sathwik-0/runtime-os](https://github.com/Sathwik-0/runtime-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
