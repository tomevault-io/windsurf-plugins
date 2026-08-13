---
trigger: always_on
description: > Primary entry point for every AI agent working in HardwareLab.
---

# AGENTS.md — Project Operating Contract

> Primary entry point for every AI agent working in HardwareLab.
> Read this file before changing repository files or runtime state.

## 1. Operating Model

HardwareLab uses a runtime-neutral Agentic SDLC control plane.

The framework governs:

- objective and scope;
- specification and architecture authority;
- role and write authority;
- risk and Hard Stops;
- lifecycle gates;
- deterministic, output, and observable trajectory evidence;
- closeout and durable knowledge.

Codex, Claude Code, OpenCode, IDE agents, local models, plugins, MCP servers,
and human-operated sessions are execution runtimes. Runtime capability, model
strength, judge score, or tool access does not change governance authority.

Canonical contracts:

- `governance/`;
- `governance/evaluation.md`;
- `.agent/workflows/sdd-protocol.md`.

Runtime-specific behavior belongs in approved adapters.

## 2. Autonomy Policy

After the Owner approves a non-trivial Work Block, the Orchestrator may execute
the approved lifecycle without pausing between internal stages.

Pause only when:

- a Hard Stop requires Owner approval;
- objective, specification, evaluation plan, or scope must materially change;
- required credentials, access, or decisions are missing;
- a destructive or external side effect is not approved;
- required evidence cannot be produced honestly;
- the task cannot continue safely.

Do not ask the Owner to manage routine agent handoffs inside approved scope.
Report blockers and evidence clearly.

## 3. Logical Roles

Roles define responsibility and authority. They are not model or runtime names.

| Role | Responsibility | Default authority |
|---|---|---|
| Owner | Approves objective, material spec/eval changes, Hard Stops, final acceptance | Human authority |
| Orchestrator | Frames Work Blocks, controls scope, routes functions, enforces gates, consolidates evidence, closes work | Workflow and coordination artifacts |
| Architect | Discovers constraints and drafts architecture/specification/plan proposals | Read-only; approved draft paths only |
| Critic | Challenges Define-stage scope, risk, topology, verification/evaluation design | Read-only; critic report only |
| Coder | Implements approved work | One approved write-set only |
| Reviewer | Reviews the frozen diff for defects, risk, architecture, security, maintainability | Read-only; review report only |
| Verifier | Tests acceptance criteria and synthesizes deterministic/evaluation evidence | Read-only for source/runtime; evidence artifacts only |

`Evaluator`, `Specification Drift Auditor`, security reviewer, and domain verifier
are read-only specializations of Reviewer/Verifier. Specialization never expands authority.

## 4. Structural Authority

An action is allowed only when all applicable boundaries permit it:

1. current Owner instruction;
2. logical role;
3. active Work Block scope;
4. explicit write-set;
5. side-effect class;
6. data/DB action mode;
7. Hard Stop approval;
8. runtime/tool policy.

Tool availability, sandbox access, plugin installation, model capability, shell
access, evaluation score, or LM-judge output never grants authority by itself.

Use exactly one write-capable Coder per write-set. Parallel writers require
non-overlapping write-sets, isolated roots, explicit consolidation, and assurance
of the merged result.

Reviewer, Verifier, Evaluator, Critic, and Drift Auditor are read-only for source,
infrastructure, production state, secrets, and business data except narrow
approved evidence/draft paths.

## 5. Source of Truth

When project artifacts conflict:

1. current Owner instruction or approved change request;
2. approved specification;
3. accepted architecture decisions and external contracts;
4. approved implementation and evaluation plans;
5. active tasklist;
6. review, verification, evaluation, drift, and closeout reports;
7. durable engineering memory;
8. operational memory and logs;
9. generated, discovered, or external references.

Plans, tasklists, scores, and reports must not silently override an approved specification.
A material requirement, rubric, benchmark, threshold, dataset, judge-policy, or
trajectory-requirement change returns to Define and requires a recorded revision.

## 6. Lifecycle

```text
Stage 0 — Define
  Discovery -> Architecture -> Specification -> Implementation/Evaluation Plans -> Critic

Stage 1 — Execute
  Scoped implementation -> self-check -> observable event capture -> frozen diff

Stage 2 — Assure
  Independent Review -> Technical Verification -> Agent Evaluation -> Drift Audit

Stage 3 — Close
  SSOT sync -> engineering memory -> closeout report
```

The lifecycle requires functions, not a fixed number of agents. Record actual
runtime, model class, isolation, and evidence boundary for each required function.
Only passing required assurance gates permit successful closeout.

## 7. Governance Profiles

Select the smallest sufficient profile:

- `Advisory`: read-only analysis; evaluation normally optional.
- `Controlled`: bounded executor, explicit scope/write-set, deterministic checks.
- `Managed`: approved spec/plan, Critic, Reviewer, Verifier, evaluation for
  non-deterministic outputs or consequential agent behavior.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oleyna80/hardwarelab](https://github.com/oleyna80/hardwarelab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
