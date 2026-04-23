---
trigger: always_on
description: Orchestrates the workflow for Level 3 (Intermediate Feature) tasks, guiding AI through comprehensive planning, creative design, structured implementation, reflection, and archiving by fetching specific L3 and Core rules.
---

# LEVEL 3 WORKFLOW: INTERMEDIATE FEATURE DEVELOPMENT (AI Instructions)

> **TL;DR:** This rule orchestrates the structured workflow for Level 3 (Intermediate Feature) tasks. It guides the AI through comprehensive planning, targeted creative design, systematic implementation, in-depth reflection, and feature-specific archiving by fetching appropriate L3 and Core rules.

This workflow is typically fetched after VAN mode has confirmed the task as Level 3.

## 🧭 LEVEL 3 WORKFLOW PHASES (AI Actions)

### Phase 1: INITIALIZATION (Confirmation & Context)
1.  **Acknowledge & Confirm L3:**
    a.  State: "Initiating Level 3 Workflow for [Feature Name from activeContext.md]."
    b.  `read_file memory-bank/tasks.md` and `memory-bank/activeContext.md` to confirm task is Level 3 and gather initial scope.
2.  **Core Setup Verification (If not fully done by VAN):**
    a.  Ensure platform awareness: `fetch_rules` for `.cursor/rules/isolation_rules/Core/platform-awareness.mdc`.
    b.  Ensure Memory Bank structure: `fetch_rules` for `.cursor/rules/isolation_rules/Core/file-verification.mdc`.
3.  **Task Entry & Context:**
    a.  Verify `tasks.md` has an entry for this L3 feature.
    b.  `edit_file memory-bank/activeContext.md` to set focus: "Focus: L3 Feature - [Feature Name] - Initializing."
4.  **Milestone:** State "L3 Initialization complete. Proceeding to Documentation Setup."

### Phase 2: DOCUMENTATION SETUP (L3 Specific)
1.  **Update `projectbrief.md` (Briefly):**
    a.  `read_file memory-bank/projectbrief.md`. Use `edit_file` to add a note if this L3 feature significantly impacts overall project goals.
2.  **Update `activeContext.md`:**
    a.  Use `edit_file` to set `memory-bank/activeContext.md` focus: "Current Focus: Planning Level 3 Feature - [Feature Name]".
3.  **Prepare `tasks.md` for L3 Planning:**
    a.  Acknowledge that `tasks.md` will be updated extensively in the next phase.
4.  **Milestone:** State "L3 Documentation Setup complete. Proceeding to Feature Planning."

### Phase 3: FEATURE PLANNING (PLAN Mode Actions)
1.  **Fetch L3 Planning Rules:**
    a.  State: "Fetching Level 3 comprehensive planning and task tracking guidelines."
    b.  `fetch_rules` for `.cursor/rules/isolation_rules/Level3/planning-comprehensive.mdc`.
    c.  (The `planning-comprehensive.mdc` rule will internally reference the structure from `Level3/task-tracking-intermediate.mdc` for `tasks.md` updates).
2.  **Follow Fetched Rule (`planning-comprehensive.mdc`):**
    a.  This rule will guide you to use `edit_file` to update `memory-bank/tasks.md` with:
        *   Detailed feature description, goals, requirements (functional & non-functional).
        *   Component analysis (new, modified, interactions).
        *   Implementation strategy and high-level steps.
        *   Dependencies, risks, and mitigations.
        *   **Crucially: Flag aspects requiring CREATIVE mode.**
        *   Testing strategy overview.
3.  **Update Context & Recommend Next Mode:**
    a.  `read_file memory-bank/tasks.md` to see if any "CREATIVE: ..." items were flagged.
    b.  Use `edit_file` to update `memory-bank/activeContext.md`: "Planning complete for L3 feature [Feature Name]. Creative phases [identified/not identified]."
    c.  **If CREATIVE phases flagged:** State "Level 3 Planning complete. Creative design phases identified in `tasks.md`. Recommend CREATIVE mode." Await user.
    d.  **If NO CREATIVE phases flagged:** State "Level 3 Planning complete. No specific creative design phases flagged. Recommend IMPLEMENT mode (or VAN QA if complex tech setup anticipated)." Await user.
4.  **Milestone:** Planning phase complete. Await user confirmation for next mode.

### Phase 4: CREATIVE PHASES (CREATIVE Mode Actions - If Triggered)
1.  **Acknowledge & Fetch Creative Orchestrator:**
    a.  State: "Initiating CREATIVE mode for L3 feature [Feature Name] as per plan."
    b.  `fetch_rules` for `.cursor/rules/isolation_rules/visual-maps/creative-mode-map.mdc`.
2.  **Follow Fetched Rule (`creative-mode-map.mdc`):**
    a.  This rule will guide you to:
        *   Identify "CREATIVE: Design..." sub-tasks from `tasks.md`.
        *   For each, fetch the appropriate `Phases/CreativePhase/[design-type].mdc` rule.
        *   Generate design options, make decisions, and document in `memory-bank/creative/creative-[aspect]-[date].md` using `edit_file`.
        *   Update `tasks.md` to mark creative sub-tasks complete and link to documents.
3.  **Update Context & Recommend:**
    a.  Use `edit_file` to update `memory-bank/activeContext.md`: "Creative design phases complete for L3 feature [Feature Name]. Ready for Implementation."
    b.  State: "Level 3 Creative phases complete. Design documents created. Recommend IMPLEMENT mode."
4.  **Milestone:** Creative phase complete. Await user confirmation for IMPLEMENT mode.

### Phase 5: IMPLEMENTATION (IMPLEMENT Mode Actions)
1.  **Fetch L3 Implementation Rule:**
    a.  State: "Initiating Implementation for L3 feature [Feature Name]."
    b.  `fetch_rules` for `.cursor/rules/isolation_rules/Level3/implementation-intermediate.mdc`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Faturrachman-dev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
