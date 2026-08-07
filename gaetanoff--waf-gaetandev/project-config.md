---
trigger: always_on
description: > Entrypoint for OpenCode, GitHub Copilot, and all agent-compatible tools.
---

# AGENTS.md — Bootstrap IA: Spec Driven Development

> Entrypoint for OpenCode, GitHub Copilot, and all agent-compatible tools.
> Specs are the single source of truth.
> Detailed rules are in `.cursor/rules/core-*.mdc`, `global-*.mdc`, `specific-*.mdc`.

---

## Quick Start — Choose Your Mode

```
New project (no existing code)?
  → Use Greenfield mode: templates/workflows/greenfield.md

Existing project (adding feature or fixing bug)?
  → Use Legacy mode: templates/workflows/legacy.md

Adding a feature to a spec-covered project?
  → Use Feature Addition: templates/workflows/feature-addition.md

Fixing a bug?
  → Use Bug Fix workflow: templates/workflows/bug-fix.md
```

---

## SDD Phase Reference

| Phase | What Happens | Gate to Pass |
|---|---|---|
| 0 — Discovery | 5 questions, actors, constraints, mission.md | All unknowns resolved |
| 1 — Specification | OpenAPI, JSON Schema, Gherkin — status: draft → approved | Spec review approved |
| 2 — Architecture | ADRs, C4 diagrams, data model | ADRs accepted |
| 3 — Planning | Epics → slices → tasks with spec references | Slices defined |
| 4 — Scaffolding | Project structure, tooling, stubs from specs | spec:lint passes |
| 5 — Implementation | Types → migration → failing test → code | Conformance tests pass |
| 6 — Gates | spec:lint, typecheck, conformance, behavior, security, perf, PR checklist | All 7 gates pass |
| 7 — Iteration | Spec change → version bump → code → gate → changelog → release | Release ready |

---

## Critical Invariants — Non-Negotiable

1. **No code before `approved` spec** — always write spec first, get it approved, then implement
2. **No endpoint without OpenAPI contract** — every route has an operationId in the spec
3. **No entity without JSON Schema** — every data shape has a schema with `additionalProperties: false`
4. **No critical behavior without Gherkin** — happy path + 2 error paths minimum
5. **No breaking change without MAJOR bump** — removing fields, renaming, changing types = MAJOR
6. **No breaking change without ADR** — every breaking change has a decision record
7. **No PR without gate check** — all 7 gates in the PR checklist signed off
8. **No vague request → code** — trigger discovery protocol, ask the 5 questions first
9. **Spec wins when code and spec disagree** — fix the code or explicitly update the spec through review
10. **No silent spec changes** — every spec modification is explicit and versioned

---

## Anti-Vibe Coding Protocol

When a request is vague, incomplete, or missing success criteria:

```
Stop. Ask the 5 discovery questions:

1. WHO — What actor triggers this? What system receives the result?
2. WHAT — What is the precise input → transformation → output?
3. WHEN — What conditions trigger this? What are the edge cases?
4. WHY WRONG — What does failure look like? What errors must be handled?
5. DONE — How do we verify it works? What is the acceptance criterion?

Do not write any spec or code until all 5 are answered.
```

---

## Spec Lifecycle

```
draft → reviewed → approved → implemented → validated → deprecated
```

- **`draft`**: No implementation. Spec is being written.
- **`approved`**: Implementation may start. Contract is binding.
- **`implemented`**: Code written. Awaiting conformance validation.
- **`validated`**: All gate checks pass. Release candidate.
- **`deprecated`**: Sunset in progress. Migration guide must exist.

---

## Quality Gates Summary

| Gate | Tool | Blocks |
|---|---|---|
| G1 — Spec lint | `spectral lint` | Merge |
| G2 — Type check | `tsc --noEmit` | Merge |
| G3 — API conformance | `dredd` / `prism` | Merge |
| G4 — Behavior tests | `cucumber-js` | Merge |
| G5 — Security | `npm audit` + SAST | Merge |
| G6 — Performance | `k6` vs SLO thresholds | Release |
| G7 — PR checklist | Human review | Merge |

---

## Spec Templates

Ready-to-use templates in `templates/specs/`:

| Template | Use For |
|---|---|
| `mission.md` | Project problem statement and goals |
| `product-brief.md` | Business context and success metrics |
| `requirements.md` | Functional and non-functional requirements |
| `architecture.md` | C4 diagrams, data model, module structure |
| `api.openapi.yaml` | REST API contracts (OpenAPI 3.1) |
| `schema.json` | JSON Schema data contracts |
| `gherkin.feature` | Behavior specifications |
| `decision-record.md` | Architecture Decision Records |
| `plan.md` | Implementation plan with slices and tasks |
| `tasks.md` | Sprint task list with spec references |
| `validation.md` | Gate check and release validation report |
| `changelog.md` | Keep a Changelog format |

---

## Prompt Library

### Discovery
```
"I want to [feature]. Before any spec or code, ask the 5 SDD discovery questions."
```

### Write Spec
```
"Write [spec type] for [feature] at specs/[path].
Use the template from templates/specs/[template]. Status: draft.
Do not write any implementation code."
```

### Implement a Slice
```
"Implement [feature] against:
@specs/api/[domain].openapi.yaml (operation: [operationId])
@specs/schemas/[entity].schema.json
@specs/features/[feature].feature
Follow SDD order: types → migration → failing test → data → logic → controller."
```

### Gate Check
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GaetanOff/WAF-GaetanDev](https://github.com/GaetanOff/WAF-GaetanDev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
