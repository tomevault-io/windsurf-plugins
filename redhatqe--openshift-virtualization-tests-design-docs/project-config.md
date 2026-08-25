---
trigger: always_on
description: This document defines the review standards for Software Test Plans (STPs) in this repository.
---

# AI Review Guide for STP Pull Requests

This document defines the review standards for Software Test Plans (STPs) in this repository.
It is used by AI review tools (CodeRabbit, Claude Code) and human reviewers.

For template structure and STP lifecycle, see `docs/stp-guide.md` and `stps/stp-template/stp.md`.

Assisted-by: Claude <noreply@anthropic.com>

---

## Core Principles

1. **User perspective only** — STPs describe what users experience, not how the system works internally.
   No API field names, CRD names, internal component references, or implementation mechanisms.
2. **Every claim needs evidence** — sign-offs, Jira links, dates. No empty placeholders in approved STPs.
3. **Distinguish constraint categories** — Feature Limitations, Test Limitations, Out of Scope, and Risks
   are four distinct concepts. Never mix them.
   See `docs/stp-guide.md` Section II.8 (Constraints Summary) for definitions.
4. **Traceability is mandatory** — every test scenario must map to a Jira requirement ID with tier and priority.
5. **Concise and actionable** — no walls of text, no template boilerplate left in, no vague statements.

---

## Section-by-Section Review Checklist

### Metadata & Tracking

- [ ] Enhancement(s) links to a VEP, design doc, or HLD (not "N/A" without justification)
- [ ] Feature Tracking links to the feature-level Jira
- [ ] Epic Tracking links to the feature tracking epic (not the QE Jira)
- [ ] Feature Maturity lists each phase with its target version using the structured format:
  `DP: [version or N/A]`, `TP: [version or N/A]`, `GA: [version]`.
  Standard maturity phases: **Dev Preview (DP)**, **Tech Preview (TP)**, **General Availability (GA)**.
  A typical progression is DP → TP → GA across releases (e.g., DP in 4.22, TP in 4.23, GA in 5.0).
  For multi-phase features, the STP scope must clearly state which phase it covers.
- [ ] QE Owner(s) listed with name and contact
- [ ] Owning SIG and Participating SIGs are correct
- [ ] Document Conventions defines only feature-specific terms, not standard ones
  (VM, PVC, CDI, etc. are known to all reviewers — do not define them)
- [ ] Document Conventions uses a bulleted list with one term per line, not inline comma-separated text
- [ ] Reviewer should follow the linked feature request and tracking epic
  to verify:
  - Requirements in the STP align with the feature request definition
  - Acceptance criteria cover the scope defined in the feature epic
- [ ] VEP and design doc links are present and the STP content
  is consistent with those sources

### Feature Overview

- [ ] 2-8 sentences maximum
- [ ] Describes what the feature does from the user's perspective
- [ ] Explains why it matters to customers
- [ ] No implementation details (no API names, no internal component names)
- [ ] For multi-phase features (Dev Preview → Tech Preview → GA), states the current
  phase and which phase this STP covers
- [ ] Claims made in the Feature Overview (e.g., "isolated," "secure," "non-disruptive,"
  "seamless") must have matching acceptance criteria and test scenarios. If the feature
  doesn't test a claim: remove the claim from the Overview, document it in Out of Scope
  with Rationale and PM/Lead Agreement (name/date), or reference existing coverage
  elsewhere (do not use Out of Scope for work already covered by other teams/suites).

**Common rejection reasons:**
- Feature Overview makes claims ("hardware-isolated," "safe multi-tenant") with no matching
  acceptance criteria or test scenarios, and without documenting the gap in Out of Scope
  with Rationale and PM/Lead Agreement
- Feature Overview describes internal system changes instead of user capabilities

### I.1 — Requirement & User Story Review

- [ ] All checklist items have checkboxes marked `[x]` with content filled in
- [ ] Requirements are listed as specific, testable items — not a repetition of the Feature Overview
- [ ] Customer use cases are in user story format ("As a [role], I want to [action]")
- [ ] Acceptance criteria are individual list items — each is a specific, verifiable pass/fail condition
- [ ] Acceptance criteria describe observable user outcomes, not internal system behavior
- [ ] For features claiming seamless or non-disruptive behavior, acceptance criteria must include
  at least one condition that can *only* pass if disruption never occurred. A criterion that checks
  end-state alone (e.g., "X is present after the operation") is insufficient — it would also pass
  after a disruptive stop-and-restart cycle. The criterion must be one that *would fail* if a
  disrupt-then-restore sequence happened instead of the claimed seamless path.
- [ ] NFRs explicitly address: Monitoring, Observability, UI, Documentation, Performance, Security, Scalability
- [ ] NFRs not covered have justification
- [ ] Scalability NFR: if the feature relies on a platform mechanism (e.g., live migration) that
  has existing scale constraints (e.g., cluster-level parallelism limits), those constraints must
  be acknowledged — even if the feature itself introduces no new scale requirements. Saying "no new
  scale requirements" is not sufficient when the underlying mechanism imposes limits.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RedHatQE/openshift-virtualization-tests-design-docs](https://github.com/RedHatQE/openshift-virtualization-tests-design-docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
