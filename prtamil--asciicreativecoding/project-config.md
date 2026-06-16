---
trigger: always_on
description: Behavioral guidelines to reduce common LLM coding mistakes.
---

# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## Modes & Phases

A file in this codebase passes through three phases. Each activates a different mode with different rules — when rules appear to contradict, the active mode wins.

| Phase / Mode | Trigger | Goal | Rules dominate |
|---|---|---|---|
| **Phase 1 — Author** (new file) | "write a new X", "add a demo for Y" | working production version, validated by clean compile + visual inspection. 250-450 lines. | `documentation/Authoring.md`; *New Simulation Workflow* below |
| **Phase 2 — Iterate** (surgical edit) | bug fix, "change X to Y", visual symptom report | minimum diff that solves the request | §3 *Surgical Changes* |
| **Phase 3a — DEFAULT_REFACTOR** (default for ~80 % of files) | `DEFAULT_REFACTOR` / `/default-refactor <file>`, or *refactor* on any file where the reader is a competent programmer (knows C, may not know the domain) | concept density + pseudocode-first; Tier-3 on data structures + drivers; light everywhere else | `default-refactor` skill (`.claude/skills/default-refactor/SKILL.md`); overrides "match existing style" and the 250-450 target |
| **Phase 3b — NOVICE_REFACTOR** (textbook for true beginners) | `NOVICE_REFACTOR` / `COMPLETE_LITERATE` / `LITERATE` (legacy aliases) / `/novice-refactor <file>`, or *refactor* on a CANONICAL algorithm reference where the reader has no domain background | embedded textbook with analogies, worked examples, GUIDED TUTORIAL with construction-sequence lessons | `novice-refactor` skill (`.claude/skills/novice-refactor/SKILL.md`); reserve for the 5–10 canonical references |
| **Phase 3c — COMMENT_REFACTOR** (comment-only) | `COMMENT_REFACTOR` / `UPDATE_LITERATE` (legacy alias) / `/comment-refactor <file>`, "redo the comments on X" | rewrite the comment layer from zero against a code-only inventory; defaults to DEFAULT profile, pass `novice` arg to override; code untouched | `comment-refactor` skill (`.claude/skills/comment-refactor/SKILL.md`) |

When unsure which mode applies, name it explicitly: *"This is a Surgical edit, so I'll only touch X."*

A surgical edit (bug fix, theme add) does NOT move a file between phases — a phase-3 textbook can take a phase-2 fix without losing textbook status.

### Phase 1 — Author

**Produce:** file header, CONCEPTS, MENTAL MODEL, §1..§N code following framework.c, HUD. Themes only if requested. Full templates and examples in `documentation/Authoring.md`.

**Do NOT:** add HOW TO READ THIS FILE, GUIDED TUTORIAL, debug overlays, long-name expansion, or per-function teaching blocks. Pedagogy is phase 3's job.

### Phase 2 — Iterate

User runs the program, reports what they see; converge via surgical edits. One concern per turn. No restructuring beyond the request, no "while we're here".

**End** = explicit user approval ("looks good" / "ship it"). The file is then **validated**.

### Phase 3a — DEFAULT_REFACTOR (the default for ~80 % of files)

Trigger words `DEFAULT_REFACTOR` / `/default-refactor <filename>` auto-invoke the `default-refactor` skill. Same 3-step structure as the others (Step 0 themes/HUD, Step 1 pseudocode-shaped code, Step 2 prose layer), but Step 2 is built for a **competent learner** (knows C, knows general programming, may not know the domain). Step 2 produces SEVEN pieces in fixed order: ABSTRACT (3-5 sentences), SECTION MAP, CONCEPTS & ALGORITHMS (dense bullet list with refs), OVERALL PSEUDOCODE (whole program in 10-20 lines), DRIVER PSEUDOCODE (per driver, top-of-file), KEYS+BUILD, plus inline Tier-3 above each main DATA STRUCTURE and DRIVER function. Tier-1 one-liners only on algorithmic helpers; nothing on plumbing. **Concept density + pseudocode-first** instead of analogies and worked examples.

### Phase 3b — NOVICE_REFACTOR (textbook for true beginners)

Trigger words `NOVICE_REFACTOR` / `COMPLETE_LITERATE` / `LITERATE` (legacy aliases) / `/novice-refactor <filename>` auto-invoke the `novice-refactor` skill. The **textbook** treatment: Step 2 produces file header + CONCEPTS + MENTAL MODEL (with worked example + ASCII diagram) + GUIDED TUTORIAL (4–6 PROBLEM/FIX/WHY+WHERE lessons) + Tier-1 function labels. Reserve for the **5–10 canonical algorithm references** in the project where a true beginner with no domain background needs to learn from zero.

### Phase 3c — COMMENT_REFACTOR (comment-only refresh)

Trigger word `COMMENT_REFACTOR` (or legacy `UPDATE_LITERATE` / `/comment-refactor <filename>`) auto-invokes the `comment-refactor` skill. Comment-only sibling of both DEFAULT and NOVICE — assumes Step 0 and Step 1 are already done; rewrites only the prose layer. **Defaults to DEFAULT_REFACTOR profile silently**; pass `novice` as an arg (`/comment-refactor novice <file>`) to target NOVICE instead.

### Phase violations

- DEFAULT_REFACTOR / NOVICE_REFACTOR requested before phase 2 ended → ask whether the algorithm is validated. Pedagogy on a buggy algorithm wastes effort.
- COMMENT_REFACTOR requested on a file whose code is NOT pseudocode-shaped (no helpers, no named locals, no layer functions) → push back: run `default-refactor` (or `novice-refactor`) Step 1 first, then COMMENT_REFACTOR.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prtamil/AsciiCreativeCoding](https://github.com/prtamil/AsciiCreativeCoding) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
