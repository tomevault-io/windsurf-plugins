---
trigger: always_on
description: This file is pre-loaded by every agent working on this repo. It is the single source of truth for how to work here. The workflow is the **G0–G6 stage-gated pipeline** (§2): every unit of work passes every gate, and every gate is a checkable artifact — not a vibe.
---

# AGENTS.md — Runtime instructions for every agent in Audit-skills

This file is pre-loaded by every agent working on this repo. It is the single source of truth for how to work here. The workflow is the **G0–G6 stage-gated pipeline** (§2): every unit of work passes every gate, and every gate is a checkable artifact — not a vibe.

---

## 0. Before you do anything

### 0.1 Read `docs/lessons-learned.md`
That file captures every mistake we have made across 3 build waves and 3+ rounds of verification. The short version:
- **Never trust your own recall for factual claims.** Verify against live sources via `webfetch`.
- **Counts are always wrong by ±2.** Verify every count against the source document.
- **Every manifest URL must be live-checked.** URL rot is the #1 silent failure mode.
- **Build agents produce plausible-sounding fabricated identifiers.** The §5.11 verification gate is the only guardrail.

### 0.2 The ratchet rule
Every entry in `docs/lessons-learned.md` must map to a linter rule, a test, or a CI job. If you learn a new lesson, add it to lessons-learned **and** open a Linear ticket to mechanize it. A lesson that only lives in prose is not learned.

### 0.3 Know the repo structure
- `skills/<slug>/SKILL.md` — router, ≤300 lines, frontmatter + all 12 sections
- `skills/<slug>/chunks/NN-slug.md` — content chunks, ≤200 lines each
- `skills/<slug>/industries/` — 3-4 industry view files + `_index.md`
- `skills/<slug>/use-cases/` — 3-4 use cases + `_index.md`
- `skills/<slug>/tests/` — 6 skill-specific test files (oracle, grounding, trace, metamorphic, adversarial, telemetry) + `<slug>_stub.py`; lint + consistency run from root `tests/` parametrized over all skills
- `skills/<slug>/data/` — generators, seeds, crosswalks
- `skills/<slug>/docs/` — architecture, limits, changelog, acceptance-gate
- `skills/<slug>/telemetry/` — schema, instrument, redaction, baseline
- `data/registry/citations.json` — canonical citation registry: every §10 manifest label + URL. Add citations HERE first; manifests copy the registry URL verbatim (enforced by `tests/test_citation_registry.py`)
- `prompts/` — version-controlled agent prompts for every G4 review pass (use these verbatim; do NOT improvise review instructions)
- `tools/lint_skill.py` — Tier 0a linter (G3 gate)
- `tools/check_fact_sheet.py` — fact-sheet completeness checker (G1 gate)
- `tools/check_design_doc.py` — design-doc section checker (G2 gate)
- `tools/check_link_rot.py` — nightly URL liveness
- `tests/test_consistency_lib.py` — cross-document consistency checks
- `docs/fact-sheet-template.md` — Day 0 research template (with machine-readable data block)
- `docs/skill-design-template.md` — design doc template (15 sections)

---

## 1. Work types

Every change belongs to exactly one work type. All four run the same G0–G6 spine; they differ only in which G1–G4 artifacts are required.

| Work type | G1 Research | G2 Design | G3 Build | G4 Verify |
|---|---|---|---|---|
| **A. New CORPUS skill** | Fact-sheet (full) | 15-section design doc + file-requirements spec | Router + chunks + industries + UCs + 6 test files | 5-lens + §5.11 + persona vetting + consumer smoke test |
| **B. Skill edit** | Verify changed facts vs live sources; new/changed citations go into `data/registry/citations.json` first | — (PR description suffices) | Edited files | §5.11 on changed claims; re-run affected UC smoke test if routing/UC content changed |
| **C. ARGUS tool** (audit-skills-mcp repo) | Fact-sheet (formulas, authoritative params) | Golden reference cases (SOX-612 format) | Implementation + unit tests | Harness validation vs golden cases (Epic 6) |
| **D. GTM post** | Source artifact must exist in repo | — | Post draft in `claude-outputs/` | Artifact link resolves; claims match shipped skill content |

---

## 2. The G0–G6 pipeline

### G0 — Intake
- Every unit of work starts from a Linear ticket (SOX-NNN). No ticket → create one before starting.
- Branch from `main`: `<type>/SOX-NNN-short-slug` (e.g. `feat/SOX-567-iso-27001-build`).
- The eventual PR description must contain `Closes SOX-NNN` (or `Part of SOX-NNN` for partial work) so Linear transitions automatically.

### G1 — Research (Day 0)
- Copy `docs/fact-sheet-template.md` → `docs/<slug>-fact-sheet.md`.
- A research agent **with webfetch access** populates it: every identifier, every count, every crosswalk row, every URL (live-checked, status recorded), version/supersession info, exact terminology.
- The fact-sheet's **machine-readable data block** (§0 of the template) must be filled — it is what G3 tests assert against.
- **Gate:** `python3 tools/check_fact_sheet.py docs/<slug>-fact-sheet.md` passes. Do not proceed until it does. If a build agent later needs a fact that isn't in the fact-sheet, the fact-sheet is incomplete — return to G1.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amurthygithub/Audit-skills](https://github.com/amurthygithub/Audit-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
