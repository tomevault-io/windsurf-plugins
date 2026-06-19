---
trigger: always_on
description: Primary: Markdown Release Readiness Report. Secondary: JSON summary for CI/audit pipelines.
---


# SAP Transport Gate Skill

AI-assisted pre-release review for SAP Transport Requests. Based on provided evidence, produces a structured Release Readiness Report and an auditable GO / CONDITIONAL_GO / NO_GO / NEED_MORE_EVIDENCE decision.

**Core principle**: AI does NOT log in to SAP, does NOT execute transports, and does NOT fabricate conclusions from insufficient evidence. Every finding must trace to real evidence. Every gap must be declared.

---

## Hard Constraints (enforce at all times)

| Constraint                   | Rule                                                                                          |
| ---------------------------- | --------------------------------------------------------------------------------------------- |
| No SAP login                 | Never request or accept SAP passwords, connection strings, or session tokens                  |
| No transport execution       | Never trigger transport release, import, delete, or rollback                                  |
| No evidence fabrication      | No assumption that unread objects are safe; no invented test results                          |
| Evidence Level LOW → no GO   | If Evidence Level is `LOW`, the decision must be `NEED_MORE_EVIDENCE` or `NO_GO` — never `GO` |
| Single-file ≠ full TR review | A single ABAP file does not constitute a Transport Request-level release review               |
| Missing spec → Inferred only | Without a functional specification, Functional Alignment must be labeled `Inferred / Limited` |

Violations of these constraints invalidate the review output. Check them before generating any decision.

---

## Step 0 — Load References

Before reading user-provided materials, load the following references in order:

1. `references/review-modes.md` — Mode detection, Review Package structure, evidence completeness rules
2. `references/decision-policy.md` — Evidence Level rules, Finding taxonomy, Release Decision Policy
3. `references/review-dimensions.md` — 10 review dimensions with detailed checks
4. `references/report-format.md` — Markdown report template and JSON schema
5. `references/abap-security-rules.md` — Full SEC-_ and AUTH-_ security rule library (load for Dimension 3 and 4)
6. `references/abap-quality-rules.md` — Full Clean ABAP quality rule library (load for Dimension 1 and 2)

Load `references/sap-connectivity.md` when:

- Review mode is **Online Transport Mode** (TR ID provided, no pre-exported package)

Load `references/human-loop.md` when:

- Target stage is `PRD`
- Any `HIGH` or `CRITICAL` finding is present
- Functional Alignment depends on business interpretation

Load `references/regression-tests.md` only if asked to run regression checks or build eval cases.

---

## Step 0.5 — Extract Transport Request ID

Scan the user's input for a Transport Request ID **before proceeding**. TR IDs follow the pattern: **3–4 uppercase letters + K + 6 digits** (e.g., `DEVK900123`, `ECDK943668`, `NPLK000045`).

| Case                                     | Action                                                                                                                                 |
| ---------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| TR ID found in input                     | Record it as `{TR_ID}`. Use it in the report filename, header, and all references throughout this review.                              |
| User provides a Review Package directory | Extract TR ID from `manifest.json` → `meta.tr_id` field.                                                                               |
| No TR ID found but review was requested  | Ask: _"Please provide the Transport Request ID (e.g., DEVK900123) to begin the review."_ Do not proceed until a TR ID is supplied.     |
| User confirms no TR ID exists            | Declare scope as `Partial / No TR Identity`. Evidence Level is capped at `LOW`. Proceed only to produce a `NEED_MORE_EVIDENCE` report. |

The TR ID is required for the report filename, the Executive Decision header, the Appendix object list, and audit traceability.

---

## Step 1 — Identify Review Mode

Determine the review mode from the input. Full rules in `references/review-modes.md §1`.

| Signal                                                                                       | Mode                  |
| -------------------------------------------------------------------------------------------- | --------------------- |
| User provides a standard Review Package (manifest + object list + sources + evidence)        | Offline Package Mode  |
| User provides only source files or partial materials (no manifest, no full TR context)       | Offline Local Mode    |
| User provides a Transport Request ID and CLI output or live system access via internal tools | Online Transport Mode |

**Online Transport Mode — Proactive Collection Protocol:**

When a TR ID is identified and no pre-assembled package exists, the SKILL must **proactively attempt collection** rather than waiting for the user to provide materials. Execute the following in order:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shrek-abaper/sap-transport-gate](https://github.com/shrek-abaper/sap-transport-gate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
