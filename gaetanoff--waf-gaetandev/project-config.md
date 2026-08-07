---
trigger: always_on
description: Discovery phase — structured requirements gathering, anti-vibe coding protocol, clarification before implementation
---


# Core Discovery — Anti-Vibe Coding Protocol

> Discovery is the zero phase. No spec written, no architecture drawn, no code touched until discovery is complete and signed off.

---

## Anti-Vibe Coding — Mandatory Protocol

**Never start coding from a vague request.** Vibe coding produces code that cannot be tested, maintained, or validated against any contract. It creates technical debt from line 1.

### Trigger Conditions — Stop and Clarify

Stop immediately and initiate discovery when:

- The request uses vague terms: "do something with", "make it better", "add a feature", "fix the issue", "improve performance"
- The domain model is undefined or implicit
- Success criteria are missing or unmeasurable
- The target user or actor is not named
- The request contradicts an existing spec without acknowledging it
- Edge cases are not mentioned for a business-critical flow
- The scope is unbounded ("handle all cases", "support everything")

### Minimum Viable Clarification — The 5 Questions

Ask exactly the questions that unblock spec writing. Never more, never less. Group them in a single message.

```
Before I write the spec and start implementation, I need 5 answers:

1. WHO — What actor/user triggers this? What system receives the result?
2. WHAT — What is the precise expected outcome? (input → transformation → output)
3. WHEN — What conditions/events trigger this flow?
4. WHY WRONG — What does failure look like? What errors must be handled?
5. DONE — What does "done" mean? How do we verify it works?
```

**Do not ask for implementation preferences** ("should I use REST or GraphQL?"). That is architecture, not discovery.

### Discovery Output — Required Artifacts

Discovery is complete only when these 3 artifacts are drafted:

| Artifact | Format | Purpose |
|---|---|---|
| `specs/mission.md` | Markdown | Problem statement, actors, goals, non-goals |
| `specs/requirements.md` | Markdown + Given-When-Then | Functional + non-functional requirements |
| `specs/decisions/ADR-000-project-context.md` | ADR template | Initial architectural context |

---

## Structured Discovery Process

### Phase 0.1 — Problem Statement

```markdown
## Problem Statement

**Context**: [Current situation — what exists, what is broken or missing]
**Problem**: [The actual pain point, with evidence if possible]
**Impact**: [Who is affected, how often, what is the cost of inaction]
**Hypothesis**: [The proposed solution direction — not the implementation]
```

### Phase 0.2 — Actor Mapping

Identify every actor (human or system) that interacts with the feature:

```markdown
## Actors

| Actor | Type | Goal | Permissions |
|---|---|---|---|
| End user | Human | Creates and manages resources | Read, Write own |
| Admin | Human | Oversees all resources | Full access |
| Payment gateway | External system | Processes transactions | Webhook push |
| Background job | Internal system | Processes async tasks | Internal |
```

### Phase 0.3 — Constraint Inventory

Collect hard constraints before any design decision:

```markdown
## Constraints

### Technical Constraints
- [ ] Existing system integrations (APIs, databases, auth providers)
- [ ] Performance SLOs (response time, throughput, availability)
- [ ] Deployment environment (cloud provider, region, container runtime)
- [ ] Security requirements (compliance, data residency, encryption)

### Business Constraints
- [ ] Delivery deadline
- [ ] Budget / infrastructure cost ceiling
- [ ] Regulatory requirements (GDPR, HIPAA, SOC2, PCI-DSS)
- [ ] Stakeholder sign-off requirements

### Knowledge Constraints
- [ ] Team expertise gaps
- [ ] Missing domain knowledge
- [ ] External dependencies not yet evaluated
```

### Phase 0.4 — Assumption Tracker

Every assumption made during discovery becomes an explicit, tracked decision.

```markdown
## Assumptions → Decisions

| ID | Assumption | Risk if Wrong | Resolution | Status |
|---|---|---|---|---|
| A-001 | Users are authenticated via OAuth2 | Auth system incompatibility | Confirmed with auth team | ✅ Confirmed |
| A-002 | p95 latency < 200ms is acceptable | SLA violation | Pending stakeholder review | ⏳ Pending |
| A-003 | Data volume < 1M records/day | Architecture mismatch | Not yet evaluated | ❌ Unknown |
```

**Rule**: Every `❌ Unknown` assumption blocks spec approval. Resolve before proceeding.

---

## Greenfield vs Legacy Discovery

### Greenfield Project Discovery

Full discovery is required. No existing constraints to preserve.

```
Discovery Checklist — Greenfield
- [ ] Problem statement written and validated
- [ ] All actors identified and mapped
- [ ] Functional requirements listed (happy path)
- [ ] Non-functional requirements listed (performance, security, scale)
- [ ] All constraints inventoried
- [ ] All assumptions tracked
- [ ] MVP scope defined (minimal set of specs that delivers value)
- [ ] Non-goals explicitly listed
- [ ] mission.md written
- [ ] requirements.md drafted
- [ ] ADR-000 written
```

### Legacy Project Discovery

Legacy discovery adds reverse-engineering the existing system before any change.

```
Discovery Checklist — Legacy
- [ ] Existing behavior documented (what does the system currently do?)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GaetanOff/WAF-GaetanDev](https://github.com/GaetanOff/WAF-GaetanDev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
