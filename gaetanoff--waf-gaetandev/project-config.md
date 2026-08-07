---
trigger: always_on
description: Agent orchestration protocol for SDD — context injection, spec-first prompting, handoff sequences, and anti-patterns for all AI coding tools
---


# Agent Orchestration (Spec-Driven)

> Never ask an AI to "build a feature" without its specification. AI agents will immediately write code when asked. Your job is to enforce the SDD protocol before any implementation prompt is issued.

---

## Supported Environments

| Tool | Context injection | Spec reference |
|---|---|---|
| Cursor | `@specs/api/openapi.yaml` in Composer | `@` file references |
| Claude Code | `@specs/` directory | Direct file paths |
| GitHub Copilot | `#file:specs/api/openapi.yaml` | `#file` references |
| Windsurf | `@specs/` in Cascade | `@` file references |
| OpenCode / generic | Include spec content in system context | Inline or `@` |

---

## Context Injection Protocol

Before any SDD session, inject the project context into the agent. This prevents the agent from making assumptions about the project.

### Context Injection Template

```
You are working on [Project Name], a [brief description].

Current project state:
- Mode: [Greenfield | Legacy]
- Phase: [Discovery | Specification | Architecture | Implementation | Validation]
- Spec maturity: [L0 | L1 | L2 | L3 | L4]

Active specs (reference these before writing any code):
- @specs/api/*.openapi.yaml       ← API contracts
- @specs/schemas/*.schema.json    ← Data contracts
- @specs/features/*.feature       ← Behavior contracts
- @specs/decisions/               ← Architecture decisions

SDD Rules:
- Do not write implementation code before specs are approved
- If the request is vague, ask the 5 clarification questions
- If a spec gap is found, stop and document it before continuing
- Reference spec IDs in every code comment and test description
- Promote spec status after each phase completes
```

---

## SDD Prompting Sequences

Break tasks into strictly ordered prompts. **Do not combine phases into a single prompt.**

### Sequence A — New Feature (Greenfield)

```
Step 1: Discovery
─────────────────
"I want to add [feature]. Before writing any spec or code, run the discovery
protocol. Ask the 5 minimum questions (WHO, WHAT, WHEN, WHY WRONG, DONE).
Do not proceed to specification until I answer them."

Step 2: Specification
─────────────────────
"Discovery is complete. Write the specs for [feature]:
1. JSON Schema in specs/schemas/[entity].schema.json (status: draft)
2. OpenAPI paths in specs/api/[domain].openapi.yaml (status: draft)
3. Gherkin scenarios in specs/features/[feature].feature (status: draft)
Use the templates in templates/specs/. Do not write any implementation code."

Step 3: Spec Review
───────────────────
[Human reviews and approves specs — sets status: approved]

Step 4: Conformance Scaffolding
───────────────────────────────
"Specs are approved. Do not write business logic yet.
1. Generate TypeScript types from the approved schemas
2. Scaffold empty route handler and service stubs
3. Write the conformance test (expect it to fail — no logic yet)
Confirm the test fails for the right reason before continuing."

Step 5: Implementation
──────────────────────
"Now implement [feature] strictly against:
- @specs/schemas/[entity].schema.json
- @specs/api/[domain].openapi.yaml (operation: [operationId])
- @specs/features/[feature].feature (scenarios: [list])
Run the conformance test after each layer (data → logic → API).
Do not modify specs unless a gap is found — follow the spec gap protocol."

Step 6: Validation
──────────────────
"Run the full gate check: spec:lint, typecheck, test:conformance, test:behavior.
Report each gate result. If any gate fails, apply the spec-fix workflow."
```

### Sequence B — Bug Fix

```
Step 1: Bug Discovery
─────────────────────
"Bug reported: [description].
1. Find the spec that defines the expected behavior for this scenario
2. If no spec exists, write the spec of the expected behavior first (status: draft)
3. Write a failing test that reproduces the bug
Do not fix the code yet."

Step 2: Spec Verification
─────────────────────────
"Confirm the failing test accurately reflects the spec.
Apply the SDD debugging decision tree:
- Is the spec correct? → fix the code
- Is the spec missing? → write the spec, get it approved, then fix the code
- Is the test wrong? → fix the test to match the spec
Tell me which path applies before writing any fix."

Step 3: Fix
───────────
"[Path confirmed]. Implement the fix so the reproduction test passes.
Do not change the spec unless the spec itself was wrong (in which case get it approved first)."
```

### Sequence C — Legacy Refactor

```
Step 1: Retro-Spec
──────────────────
"Write retro-specs for [module/file] — describe CURRENT behavior as-is.
Do not improve or change behavior. Write:
1. JSON Schema for each entity the module processes
2. OpenAPI entries for each route (if applicable)
3. Gherkin scenarios for the most critical behaviors
Mark all as status: implemented (they describe existing behavior)."

Step 2: Gap Analysis
────────────────────
"Analyze the retro-specs against the desired behavior.
List:
- Behaviors that should change (delta specs needed)
- Behaviors that should stay (retro-specs are final)
- Missing coverage (spec debt)
Do not refactor yet."

Step 3: Delta Specs
────────────────────
"Write delta specs for the changes identified:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GaetanOff/WAF-GaetanDev](https://github.com/GaetanOff/WAF-GaetanDev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
