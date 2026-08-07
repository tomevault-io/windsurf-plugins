---
trigger: always_on
description: Spec lifecycle management — states, transitions, ownership, and enforcement rules for every specification artifact
---


# Core Spec Lifecycle

> Every specification artifact has a state. State transitions are explicit, gated, and tracked. No code ships against an unapproved spec.

---

## Spec States

```
draft ──► reviewed ──► approved ──► implemented ──► validated ──► deprecated
  │                        │              │                │
  └──────── rejected ──────┘              └── superseded ──┘
```

| State | Meaning | Who Can Set | What It Enables |
|---|---|---|---|
| `draft` | Work in progress, not ready for review | Author | Nothing — no implementation |
| `reviewed` | Reviewed, feedback addressed | Reviewer | Nothing — awaiting approval |
| `approved` | Contract is final, ready to implement | Lead / Stakeholder | Implementation may start |
| `implemented` | Code conforms to spec | Developer | Validation may start |
| `validated` | Conformance tests pass, contract verified | QA / CI | Release may proceed |
| `deprecated` | Spec is no longer active, sunset in progress | Lead | Migration required |
| `superseded` | Replaced by a newer version of the spec | Author | Old spec archived |

---

## Frontmatter Convention

Every spec file **must** include a `status` field in its YAML frontmatter:

```yaml
---
id: spec-001
title: User Authentication API
type: openapi | json-schema | gherkin | adr | pact | asyncapi
status: draft | reviewed | approved | implemented | validated | deprecated
version: 1.0.0
authors:
  - name: John Doe
    email: john@example.com
created: 2024-01-15
updated: 2024-03-20
depends_on:
  - spec-002  # User schema
supersedes: ~
---
```

**Required fields**: `id`, `title`, `type`, `status`, `version`, `authors`, `created`, `updated`

---

## Transition Rules

### `draft` → `reviewed`

Requirements to transition:
- [ ] Spec is syntactically valid (YAML/JSON lints cleanly, OpenAPI validates)
- [ ] All required sections are filled (no `TODO` or `TBD` placeholders)
- [ ] At least one example per endpoint / field / scenario
- [ ] Error cases are documented
- [ ] Author has self-reviewed the spec

Action: Open a spec review (PR, doc comment, or team review session).

### `reviewed` → `approved`

Requirements to transition:
- [ ] All review comments addressed or explicitly rejected with reason
- [ ] No open blocking questions
- [ ] Lead or designated approver has signed off
- [ ] Dependent specs are also at `approved` or `validated`
- [ ] ADR exists if the spec introduces a new architectural pattern

Action: Approver sets status to `approved`. This is a binding decision.

### `approved` → `implemented`

Requirements to transition:
- [ ] All conformance tests generated from the spec
- [ ] Code passes all conformance tests
- [ ] No deviations from spec — if a deviation was found, spec was updated first
- [ ] Code review completed with spec conformance verified

Action: Developer sets status to `implemented` in the spec file.

### `implemented` → `validated`

Requirements to transition:
- [ ] Conformance test suite passes in CI
- [ ] Integration tests pass
- [ ] Performance SLOs meet spec thresholds
- [ ] Security review passed (if spec declares security requirements)
- [ ] No spec debt items remain open

Action: CI/CD pipeline or QA engineer sets status to `validated`.

### `validated` → `deprecated`

Requirements to transition:
- [ ] Migration guide written
- [ ] Consumers notified (internal teams, external clients)
- [ ] Sunset date published
- [ ] Replacement spec exists at `validated` status
- [ ] Deprecation header added to API responses if applicable

Action: Lead sets status to `deprecated`. Start sunset clock.

---

## Spec Types and Their Artifacts

| Type | File Pattern | Tool | Validates With |
|---|---|---|---|
| OpenAPI 3.1 | `specs/api/*.openapi.yaml` | Spectral | Dredd, Prism |
| JSON Schema | `specs/schemas/*.schema.json` | AJV | Unit tests |
| Gherkin | `specs/features/*.feature` | Cucumber | Playwright, Vitest |
| ADR | `specs/decisions/ADR-*.md` | Manual review | PR approval |
| Pact | `specs/contracts/*.pact.json` | Pact Broker | Pact verifier |
| AsyncAPI | `specs/events/*.asyncapi.yaml` | Spectral | Event bus tests |
| SLO | `specs/slos/*.slo.yaml` | Custom | Alerting system |

---

## Spec Versioning

Specs follow **semantic versioning**. Version increments are governed by the change type:

| Change Type | Version Bump | Example |
|---|---|---|
| Typo fix, clarification | `patch` | 1.0.0 → 1.0.1 |
| New optional field, new endpoint | `minor` | 1.0.0 → 1.1.0 |
| Renamed field, removed field, changed type | `major` | 1.0.0 → 2.0.0 |

**Major version bumps require**:
- A new ADR documenting the breaking change
- A migration guide
- A deprecation period for the previous version

---

## Spec ID Convention

Specs are assigned a unique, stable ID at creation:

```
<type>-<domain>-<sequence>

Examples:
  api-user-001       → First user API spec
  schema-order-003   → Third order schema
  feat-checkout-007  → Seventh checkout feature spec
  adr-auth-002       → Second auth decision record
```

IDs never change, even after spec updates.

---

## Spec Directory Structure

```
specs/
├── api/                    # OpenAPI specs
│   ├── user.openapi.yaml   # status: approved
│   └── order.openapi.yaml  # status: draft

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GaetanOff/WAF-GaetanDev](https://github.com/GaetanOff/WAF-GaetanDev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
