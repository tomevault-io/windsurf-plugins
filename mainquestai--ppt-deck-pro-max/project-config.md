---
trigger: always_on
description: PPT Deck Pro Max builds product-grade PPT/HTML decks from briefs, strategy docs, product narratives, or solution materials. It manages brief, narrative, visual system, image-led build, QA, and rollback.
---


# PPT Deck Pro Max

## Overview

Use this skill to produce a high-quality PPT/HTML deck from raw materials. Do not treat the task as “convert document to slides.” Treat it as a production workflow: brief, narrative, visual system, hero pages, clean page copy, build, and QA.

Before production work, run:

```bash
python3 scripts/run_deck_pipeline.py doctor
```

Use `--production-mode expert` for high-value decks that need expert enrichment. Use `--production-mode quick` for smaller decks where the Expert Interview startup cost would outweigh the value.

## Use This Skill vs. Other Skills

Use this skill when the user needs a new deck, a major remake, or a deck that must feel product-grade, sales-ready, and visually unified.

Do **not** use this skill as the default for:

1. Small edits to an existing `.pptx`
2. One-slide fixes
3. Simple document-style presentations
4. Pure slide reading or comparison work

For those cases, prefer:

- `$pptx` for inspection and editing
- `$slides` for direct `.pptx` creation
- `$frontend-design` for visual implementation help

## Forced Workflow

Do not skip steps unless the required artifact already exists and is still valid.

Artifact validity means:

1. Freshness: it reflects the current brief, audience, output mode, and page count
2. Schema: required JSON files validate against the matching schema when a schema exists
3. Coverage: every required page or beat has a non-empty entry
4. Review state: any human approval, redaction decision, or QA status is recorded in the artifact or `slide_state.json`
5. Traceability: the artifact names its source inputs or can be regenerated from the current project files

If any item fails, refresh the artifact before moving forward.

## Production Sub-Modes

Choose a production sub-mode during Step 0 and record it in `deck_brief.md` and `slide_state.json` when those artifacts exist.

Default to `standard_deck` unless the request clearly matches a specialized sub-mode.

### `standard_deck`

Use for product intros, solution decks, strategy decks, industry point-of-view decks, and business partnership decks that will be built as editable PPTX, HTML, or a mixed deck.

Follow the main forced workflow from Step 0 to Step 8.

### `formal_bid_image_led`

Use for formal bid, tender, RFP response, technical proposal, or leave-behind decks where pages may be assembled from generated full-page images.

This sub-mode adds stricter gates for:

1. Page registry and actual PPT page mapping
2. Source-id image traceability
3. Candidate / Go / No-Go image separation
4. Final assembly naming
5. Delivery QA for page ratio, title hygiene, placeholders, internal-language leakage, and backup coverage

When this sub-mode is active, the mode-specific requirements below are mandatory in addition to the main forced workflow.

### Step 0: Classify the Task

Identify:

1. Deck type: product intro, solution, internal strategy, industry point of view, or business partnership
2. Input type: long document, page outline, reference deck, or raw request
3. Output type: `pptx`, `html deck`, or both
4. Production sub-mode: `standard_deck` or `formal_bid_image_led`

If the business target is still unclear, create `deck_brief.md` first.

### Step 1: Lock the Brief

Create or update `deck_brief.md`.

It must lock:

1. Product subject
2. Product positioning
3. Audience
4. First buying reason
5. Strongest differentiation
6. Strongest proof
7. Pilot entry point
8. Final CTA

Read `references/deck_brief_template.md` if the brief is missing or weak.

### Step 1.2: Content Governance Gate (expert / longform)

Before writing narrative pages, create the content governance artifacts:

1. `deck_source_digest.md`: source inventory, key facts, usable proof, explicit gaps
2. `deck_claim_map.json`: core claims with source pages, roles, evidence, and risk notes
3. `deck_capacity_plan.md` + `deck_capacity_plan.json`: target pages, recommended pages, and max supported pages
4. `deck_gap_registry.json`: open gaps, with `blocking: true`, `status: blocking`, or `severity: critical` for hard blockers
5. `deck_question_queue.md`: prioritized questions for the user / domain expert

The three JSON artifacts are covered by `scripts/validate_schema.py`.

Gate rule:

- Do not enter page-by-page drafting when `target_pages > max_supported_pages`
- Do not enter page-by-page drafting while any blocking gap remains open
- Do not pass this gate with empty placeholder markdown artifacts
- Quick mode may skip this gate unless the user explicitly asks for content governance

Use:

```bash
python3 scripts/run_deck_pipeline.py validate \
  --project-dir <project-dir> --output-mode pptx+html --expert-mode --content-governance
```

### Step 1.3: Longform Governance Gate (expert + standard_deck + longform)

For long decks, extend Step 1.2 before page-by-page drafting.

Required additions:

1. `deck_capacity_plan.json` must include four page-budget tiers: `conservative`, `recommended`, `extended`, `appendix_heavy`
2. Each tier must name page count and page mix: core, proof, extension, appendix

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MainQuestAI/PPT-Deck-Pro-Max](https://github.com/MainQuestAI/PPT-Deck-Pro-Max) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
