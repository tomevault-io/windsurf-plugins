---
trigger: always_on
description: This file defines the durable, reusable rules for AI-assisted embedded
---

# Embedded Development Contract

This file defines the durable, reusable rules for AI-assisted embedded
firmware work. It applies minimal implementation discipline to real embedded constraints:
make the smallest correct change, preserve safety and recovery, and prove the
result with proportionate evidence.

## Input Modes

Support two user entry modes for the same workflow.

### Simple mode

Accept the smallest useful input:

```text
Project path: <path>
Goal or observed problem: <one or two sentences>
Please investigate first; do not edit code yet.
```

Use this mode when the user is new to the project or does not know its
constraints. Discover repository facts and authoritative documentation yourself.
Do not ask the user to repeat information that can be found in the project.

### Structured issue mode

For precise or complex work, accept this compact issue shape:

```text
Goal: <observable result>
Scope: <files, modules, or boundaries in scope>
Problem: <current symptom or reason>
Reference: <existing implementation or document to reuse, if any>
Constraints: <must-not-change, resource, safety, or compatibility limits>
Acceptance: <how build, host, or target success will be observed>
```

Treat `Goal`, `Problem`, and known `Constraints` as the minimum useful fields;
the other fields may be `UNKNOWN`. Use the structured fields to reduce
exploration, not to skip investigation.

Treat structured fields in the current user request as the active task. Restate
them before investigation. Empty or `UNKNOWN` project-template fields do not
override supplied task input; do not ask again for a field already provided.

### Mode selection and escalation

- Use simple mode by default when input is sparse or the task is low risk.
- Suggest structured issue mode when the task has a shared interface, protocol,
  persistence, concurrency, generated-code boundary, or hardware risk.
- Convert simple-mode input into the structured fields after investigation;
  show missing fields and ask only questions that can change the solution, risk,
  or acceptance result.
- Keep the same implementation gates in both modes: investigate, propose the
  smallest change, obtain approval, implement, and verify.
## Minimal Interaction

A new project or task may start with only:

```text
Project path: <path>
Goal or observed problem: <one or two sentences>
Please investigate first; do not edit code yet.
```

Do not require the maintainer to know every hardware or build detail in advance.
Discover facts from the repository and authoritative documents. Ask only for
information that cannot be discovered and could change the solution, risk, or
acceptance result.

Use three interaction modes:

1. **Onboard**: inspect the project and establish its baseline.
2. **Develop**: investigate a task, propose the smallest change, obtain approval,
   implement only the approved scope, then report what must be built or tested.
3. **Close out**: use the maintainer's actual build, host, and target results to
   record evidence, remaining gaps, and final status.

## Authority And Unknowns

When information conflicts, use this order:

1. Confirmed silicon/board specification and approved product requirement.
2. Frozen requirement, interface contract, and acceptance criteria.
3. Approved design decision.
4. Current code, build configuration, and reproducible measurements.
5. Analysis notes, examples, and generic recommendations.

Separate confirmed facts, assumptions, conflicts, and unknowns. Mark unknowns as
`UNKNOWN`; never turn an assumption into a hardware fact. Investigate unknowns
that can be resolved from code or documentation. Ask the maintainer about
product intent, physical observations, ownership, permissions, or decisions
that cannot be inferred. Block a high-risk implementation when a critical fact
or affected range remains unconfirmed.

## Before Editing

1. Read `PROJECT.md`, `.ai-governance/capability-map.md` when present, the
   active task, build files, generated-code notes, and affected callers.
2. Identify generated files, user-owned files, extension points, and external
   tool configuration. Prefer user-code regions and supported extension points.
   If a generated file must change, record the generator, inputs, version,
   regeneration command, and recovery path.
3. Classify risk and freeze observable acceptance criteria when the task is
   medium or high risk.
4. Separate investigation, design, implementation, and verification. If a
   maintainer owns code-change approval, do not edit source, configuration, or
   build files until the exact scope is explicitly approved.

## Risk And Documentation

Stop at the lightest process that protects the task:

- **Low**: local, reversible, no shared interface, persistence, concurrency, or
  hardware-state change. Use the task input and a minimal check.
- **Medium**: shared interface, protocol, RTOS interaction, persistence,
  resource budget, or compatibility impact. Use the necessary requirement,
  design, task, and verification records.
- **High**: boot/reset, clock, Flash/NVM/OTA, watchdog, DMA ownership, security,
  safety, power output, irreversible state, or hardware damage potential. Freeze
  requirements and recovery before implementation; require target evidence for

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crichars/embedded-project-governance](https://github.com/crichars/embedded-project-governance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
