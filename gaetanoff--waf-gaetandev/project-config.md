---
trigger: always_on
description: Master workflow for Spec Driven Development — specifications are the single source of truth, covering greenfield and legacy modes
---


# SDD Workflow (Spec Driven Development)

> Specifications are the single source of truth. Code is a consequence of specs, not the other way around. Every phase is gated. No phase starts until the previous one is complete and signed off.

---

## Quick Reference

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         SDD WORKFLOW PHASES                                 │
├─────────────┬───────────────┬──────────────────┬───────────────────────────┤
│ Phase 0     │ Phase 1       │ Phase 2          │ Phase 3                   │
│ DISCOVERY   │ SPECIFICATION │ ARCHITECTURE     │ PLANNING                  │
│ Anti-vibe   │ OpenAPI,      │ ADRs, C4,        │ Epics, slices,            │
│ questions   │ Schema,       │ data model,      │ task breakdown,           │
│ constraints │ Gherkin       │ tech stack       │ acceptance criteria       │
├─────────────┼───────────────┼──────────────────┼───────────────────────────┤
│ Phase 4     │ Phase 5       │ Phase 6          │ Phase 7                   │
│ SCAFFOLDING │ SPEC-FIRST    │ CONFORMANCE      │ ITERATION &               │
│ Structure,  │ IMPLEMENTATION│ GATES            │ RELEASE                   │
│ tooling,    │ Tests first,  │ Validate,        │ Specs evolve first,       │
│ stubs       │ then code     │ gate checks      │ changelog, semver         │
└─────────────┴───────────────┴──────────────────┴───────────────────────────┘
```

---

## Project Mode Selection

Before starting, select the correct mode. Rules differ between modes.

### Greenfield Mode

Use when: building from scratch, no existing codebase, no legacy constraints.

```
Greenfield Workflow
─────────────────
Phase 0 → Discovery (full)
Phase 1 → Write all specs from scratch
Phase 2 → Design architecture from specs
Phase 3 → Plan implementation
Phase 4 → Scaffold from specs
Phase 5 → Implement spec by spec
Phase 6 → Validate all gates
Phase 7 → Release v1.0.0
```

### Legacy Mode

Use when: existing codebase, adding features, refactoring, or fixing bugs.

```
Legacy Workflow
──────────────
Phase 0 → Spec Audit (inventory what exists)
          ↓
Phase 0b → Write retro-specs (describe current behavior as-is)
          ↓
Phase 0c → Identify gaps (what is not yet specified)
          ↓
Phase 1 → Write delta specs (what changes)
Phase 2 → Check architecture impact (ADR if breaking)
Phase 3 → Plan migration
Phase 4 → Scaffold migration artifacts (if needed)
Phase 5 → Implement against new specs
Phase 6 → Validate conformance (retro + new specs)
Phase 7 → Release with migration guide
```

---

## Phase 0: Discovery (see `core-discovery` rule)

**Required artifacts:**
- `specs/mission.md` — problem statement, actors, goals, non-goals
- `specs/requirements.md` — functional + non-functional requirements
- `specs/decisions/ADR-000-project-context.md` — initial context

**Gate to pass:** All discovery questions answered. No open unknowns. Stakeholder sign-off.

**Anti-vibe protocol:** Never proceed to specification if:
- The request is vague, unbounded, or missing success criteria
- Actors are not named
- Edge cases are not defined for business-critical flows

---

## Phase 1: Specification (see `core-specification` and `core-spec-lifecycle` rules)

Write formal, machine-readable specs BEFORE any design or code decision.

### Spec Format by Project Type

```
What are you building?
│
├── REST API?
│   ├── OpenAPI 3.1 (mandatory) → specs/api/*.openapi.yaml
│   ├── JSON Schema for entities (mandatory) → specs/schemas/*.schema.json
│   ├── Gherkin for behavior (required for critical paths) → specs/features/*.feature
│   └── Pact for consumer-driven contracts (if external consumers) → specs/contracts/
│
├── GraphQL API?
│   ├── GraphQL SDL (mandatory)
│   ├── JSON Schema for complex inputs/outputs
│   └── Gherkin for behavior
│
├── Event-Driven / Message Queue?
│   ├── AsyncAPI (mandatory) → specs/events/*.asyncapi.yaml
│   └── JSON Schema for event payloads (mandatory)
│
├── Frontend (SPA/SSR)?
│   ├── OpenAPI for all consumed APIs (mandatory)
│   ├── Component prop types (mandatory)
│   └── Storybook stories as UI specs (recommended)
│
├── Mobile App?
│   ├── OpenAPI for consumed APIs (mandatory)
│   └── Screen flow spec (recommended)
│
└── CLI Tool?
    ├── Command spec in Markdown (flags, args, output format) (mandatory)
    └── Gherkin for behavior (recommended)
```

### Spec Rules
- Every spec starts with `status: draft` (see spec lifecycle)
- Specs must be reviewed and set to `status: approved` before implementation
- No `TODO` or `TBD` in an approved spec — replace with a tracked assumption
- All error cases must be specified, not just the happy path
- All specs must include at least one example

---

## Phase 2: Architecture (see `core-architecture` rule)

Design the system from approved specs. Never design before specs exist.

### Architecture Deliverables
- `specs/decisions/ADR-001-*.md` — architecture decisions driven by specs
- C4 model Level 1 (System Context) and Level 2 (Containers) diagrams
- Data model diagram derived from JSON Schema contracts
- Sequence diagrams for critical flows derived from Gherkin scenarios

### Architecture Rules

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GaetanOff/WAF-GaetanDev](https://github.com/GaetanOff/WAF-GaetanDev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
