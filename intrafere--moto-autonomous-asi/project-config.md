---
trigger: always_on
description: Main Architecture layout/design of the distillation/compiler portion of the two-part aggregation-distillation LLM workflow.
---


Main Architecture layout/design of the distillation/compiler portion of the two-part aggregation-distillation LLM workflow.

## Workflow Compiler Note
Compiler runs independently from aggregator (manual start via API only). Strict Markov-chain: one compiler-submitter runs, submits to validator, waits for validation result before resuming. Only 1 submission in queue at a time.

## Compile/Distillation Tool Outline

Reads aggregator database + user prompt, distills into a single coherent paper. 1 high-context submitter + 1 high-param submitter + 1 validator. Sequential workflow (no parallel submitters).

Aggregator/brainstorm database material is high-priority optional source context, not a mandatory checklist. Compiler submitters may selectively use, synthesize beyond, or depart from database material when that better serves the user's prompt and remains rigorous. Validator must not reject solely for selective non-use of database material.

**Context Anchors**:
- **Paper Anchor**: `[HARD CODED END-OF-PAPER MARK -- ALL CONTENT SHOULD BE ABOVE THIS LINE]`
- **Outline Anchor** (two lines): `[HARD CODED BRACKETED DESIGNATION THAT SHOWS END-OF-PAPER DESIGNATION MARK]` then `[HARD CODED END-OF-OUTLINE MARK -- ALL OUTLINE CONTENT SHOULD BE ABOVE THIS LINE]`

**Section Placeholders** (paper only):
- `[HARD CODED PLACEHOLDER FOR THE ABSTRACT SECTION - TO BE WRITTEN AFTER THE INTRODUCTION IS COMPLETE]`
- `[HARD CODED PLACEHOLDER FOR INTRODUCTION SECTION - TO BE WRITTEN AFTER THE CONCLUSION SECTION IS COMPLETE]`
- `[HARD CODED PLACEHOLDER FOR THE CONCLUSION SECTION - TO BE WRITTEN AFTER THE BODY SECTION IS COMPLETE]`

Placeholders are STRUCTURAL MARKERS ONLY. Submissions must never include them — any that appear will be silently stripped before validation.

**Marker Integrity System (Automatic Repair)**:
Before every `_pre_validate_exact_string_match()`, system calls `paper_memory.ensure_markers_intact()` (or `outline_memory.ensure_anchor_intact()` for outline_update). If markers were missing, they are added and document is re-fetched before validation. Mode-aware: paper operations check placeholders + anchor; outline operations check outline anchor only.

**Outline is ALWAYS fully injected (never RAGed)** into all compiler mode prompts.

**Provider Selection**: Each compiler role (validator, high-context, high-param, critique submitter) can independently use LM Studio or OpenRouter with optional host provider and LM Studio fallback.

**Aggregator RAG refresh**: Every 10 accepted aggregator submissions (not immediate like aggregator).

**Enhanced Rejection Feedback Format** (`compiler_rejection_log.py`):
- Header: "🚫 REJECTED BECAUSE: [Failure Reason]"
- `validation_stage`: pre-validation (exact string check) or LLM validation (placement context)
- Full validator reasoning + 300-char submission preview
- "WHAT TO FIX" with specific instructions per failure type
- Diagnostics: needle/haystack previews (first/last 200 chars) when exact match fails

Last 10 rejections and 10 acceptances: direct injected if fit, RAG only if too large.

---

## Phase-Based Paper Construction

**PHASE ORDER (strictly enforced):** BODY → CONCLUSION → INTRODUCTION → ABSTRACT

**Explicit completion signals**: Submitter sets `section_complete: true` when phase is done. Coordinator advances ONLY on explicit signal AND verifies section header exists via regex. Paper complete when abstract phase receives `section_complete: true`.

**Implementation**:
- Phase-specific prompt functions in `construction_prompts.py`
- Phase tracking via `autonomous_section_phase` in `compiler_coordinator.py`
- `CompilerSubmission` model includes `section_complete` field

`needs_construction=true` requires non-empty `content`/`new_string` — contradictory pattern causes infinite rejection loops.

**Section Placeholder System**:
1. First body section accepted → `paper_memory.initialize_with_placeholders()` sets up all placeholders
2. Each phase completion → `paper_memory.replace_placeholder()` replaces placeholder with validated content
3. AI sees placeholders in CURRENT DOCUMENT PROGRESS = section not yet written

**Placeholder Boundary Invariant**:
```
[ABSTRACT_PLACEHOLDER]
[INTRO_PLACEHOLDER]
II. Body Section 1        <-- Body content goes here
[CONCLUSION_PLACEHOLDER]  <-- HARD BOUNDARY: Body content NEVER crosses this
[PAPER_ANCHOR]
```
Body content is ALWAYS inserted BEFORE CONCLUSION_PLACEHOLDER. `_apply_edit()` auto-corrects: if old_string anchor falls after placeholder, automatically relocates insertion to just before it.

**Paper state is ALWAYS shown** in all construction phases. Empty paper shows "(EMPTY - no content written yet)" so model uses `operation='full_content'`.

| Phase | Paper Shown | Reason |
|-------|-------------|--------|
| BODY (first) | YES (EMPTY) | Must use full_content |
| BODY (continuation) | YES | See existing sections |
| CONCLUSION | YES | Summarize body |
| INTRODUCTION | YES | Preview body+conclusion |
| ABSTRACT | YES | Summarize entire paper |

---

## Submitter-Validator Cycle

**Outline Creation (Phase 1 — Iterative):**
1. HC submitter generates outline → validator reviews (accept/reject + feedback)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Intrafere/MOTO-Autonomous-ASI](https://github.com/Intrafere/MOTO-Autonomous-ASI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
