---
trigger: always_on
description: Enables lookup in any direction (e.g., "what GRI disclosure corresponds to
---

# Impact Vision

Impact Vision is an open-source AI-powered impact measurement and SDG alignment agent for VC and impact investment funds, built on top of OpenHarness.

Current release: **0.15.0 (Trust Infrastructure)**. The v3 roadmap
(`docs/roadmap-v3.md`) and engineering plan
(`docs/roadmap-v3-implementation.md`) describe the strategic shift toward
causal-style claims, stakeholder voice as evidence, governed AI, and an
LP-grade assurance bundle.

**v4 (Consultant-Led Product Strategy)** — see `docs/roadmap-v4.md`. v4 is an
integration / packaging wave, not a re-implementation wave: engineering rule
is that new code lives in `impact/engagements/`, `tools/impact/engagement_*`
/ `tools/impact/toc_*`, and `frontend/`, and must never fork an existing v3
module. Progress so far:

- **Wave 1 / Track 1 — Consultant Engagement Workspace** (shipped). The
  `impact.engagements` package ships 12 productised engagement bundles, a
  proposal builder, a 7-phase consultant checklist, an audit-logged
  deliverable state machine, client-type templates, and the
  `engagement_workspace` agent tool.
- **Wave 2 / Track 2 — Theory of Change + KPI framework builder** (shipped).
  `impact.engagements.toc_builder` wraps the existing v3 `toc_graph`
  renderer and the 59-concept cross-reference map into a consultant-facing
  ToC canvas, an 11-rule logic-chain validator (missing assumptions, weak
  causal links, unmeasured outcomes, risk blind spots, equity lens), and
  a multi-framework KPI generator, exposed through the `toc_builder`
  agent tool.
- **Tracks 3-10 — Integration Wave** (shipped). Eight backend modules in
  `impact.engagements` plus a consolidated `engagement_suite` agent tool
  (46 actions) covering the consultant workflow end-to-end:
  - `engagements.data_room` (Track 3) — data request packs, completeness
    scoring, exception workflow, multi-entity rollup, coaching cards.
  - `engagements.value_creation` (Track 4) — pluggable `BenchmarkProvider`,
    peer dashboard, impact risk rating, value-creation plan, business
    case + scenario engine, supply-chain hotspot ranker.
  - `engagements.reporting_studio` (Track 5) — 6 named report templates,
    approval state machine, claim review panel, executive deck outline,
    public microsite bundle, multi-audience rewrite scaffold.
  - `engagements.training` (Track 6) — training plan generator (maturity
    stage aware), 6 workshop packs, investee coaching cards, learning
    loop, readiness badges with threshold enforcement.
  - `engagements.website` (Track 7, backend-only) — 7-question diagnostic
    quiz + scoring, productised-engagement gallery, benchmark teaser,
    playbook library, privacy-preserving upload demo, GDPR/PDPA-aware
    lead capture, white-label partner metadata.
  - `engagements.copilot` (Track 8) — AI output provenance
    (`CopilotOutput` + `CopilotReviewQueue`), deterministic challenge
    mode, client-safe answer mode bound to approved evidence only,
    prefix-based meeting-note ingestion.
  - `engagements.regulatory` (Track 9) — 8 jurisdiction profiles
    (EU / UK / US / Singapore / Switzerland / Canada / Japan / Australia),
    SFDR + UK SDR classifiers, deadline calendar, regulator-facing
    narrative composer.
  - `engagements.verification_bundle` (Track 10) — BlueMark-style
    3-Pillar Verification Bundle (Mandate / Practice / Reporting) with a
    HMAC-signed assurance manifest, verifier token + expiry, verifier
    marketplace directory, assurance-ready badge.

## Core Workflow

1. User uploads a pitch deck / investment memo PDF
2. `pitch_deck_analyze` extracts text, identifies impact claims, maps to IRIS+/SDGs, runs DD checklist, auto-extracts a Company model
3. Agent presents gaps and asks the most important unanswered DD questions (with NESTA evidence levels)
4. Deeper scoring via `sdg_mapper`, `five_dimension_assess`, `gap_analysis` with sector benchmarks
5. `cross_reference` tool maps metrics across all 10 frameworks
6. Greenwashing detection (standard + EU Green Claims + UK FCA + NLP) and regulatory compliance checks
7. `impact_report` generates the final assessment (HTML with Plotly charts, XLSX, CSV, JSON)

## v3 Trust Infrastructure (since 0.15.0)

Layered on top of the v2 institutional-readiness backbone (canonical
`MetricRecord`, evidence graph, audit trail, standards registry):

- **Versioned emission factors** (`impact.emission_factors`) – multi-revision
  factor catalogue with uncertainty bands, sensitivity rollups, and
  inventory-repricing helpers.
- **Stakeholder voice as evidence** (`impact.stakeholder_voice`) – Lean Data
  templates, GDPR/PDPA-compliant `ConsentRecord`, beneficiary feedback
  quality scoring, and feedback↔claim linkage.
- **AI extraction review queue** (`impact.evidence_workflow`) – policy-driven
  review with bulk/auto decisions and audit-trail integration.
- **Verification workspace** (`impact.verification_workspace`) – read-only
  assurance-pack workspace with finding lifecycle and threaded comments.
- **LP narrative + Q&A** (`impact.lp_narrative`) – audit-friendly LP
  narratives and a Q&A workspace constrained to verified data.
- **Greenwashing reviewer** (`impact.greenwashing_reviewer`) – per-claim
  explainable review with specificity classification and severity scoring.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joejoe168168/impact-vision](https://github.com/joejoe168168/impact-vision) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
