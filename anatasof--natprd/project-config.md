---
trigger: always_on
description: Activate this skill when the user wants to create, update, validate, or review a Product Requirement Document. Triggers on mentions of PRD, product requirements, user stories, acceptance criteria, or initiative documentation, or when the user describes a feature and asks for structured documentation.
---


# PRD Maker

An interactive, guided PRD generation skill for writing production-grade PRDs.
Produces fully structured PRDs with user stories, Gherkin acceptance criteria,
section-level rules, and optional sections — all in markdown format.

---

## What This Skill Does

When activated, this skill:

1. **Interviews** the user section by section using focused questions
2. **Validates** each answer against the section rules before moving on
3. **Writes** the PRD incrementally, showing each section as it's completed
4. **Enforces** the PRD standard — no placeholders, no rule violations allowed
5. **Outputs** a complete `prd.md` file (default location: `docs/prd.md`)

The 12 core sections, 4 optional sections, scoring rubric, and per-section rules
live in the supporting files referenced at the bottom of this document. Load them
on demand when you need the full text — do not duplicate them here.

---

## How to Activate

Simply say:
- `"I want a PRD for [feature/initiative name]"`
- `"Help me write a PRD"`
- `"Create a PRD for [description]"`
- `"Review my PRD"` — to validate an existing PRD file

Claude will detect the intent and load this skill automatically. In Claude Code it can also be invoked directly with `/natprd`.

---

## User-Choice Presentation

Whenever an interview question offers a choice between **2–4 discrete, mutually-exclusive options**, present it using the `AskUserQuestion` tool so the user can click rather than type. This applies across all three modes.

The tool caps at 4 listed options and automatically adds an "Other" escape, so you can cover up to 5 outcomes (4 explicit + Other) without listing them all.

Examples that MUST use buttons:
- §2 Document Status — list the 4 most common (Draft / In Review / Approved / In Execution); "Other" covers Deprecated
- §6 Hypothesis confidence (High / Medium / Low)
- §8 MoSCoW priority (Must-have / Should-have / Could-have / Won't-have)
- §7 Metric type (Leading / Lagging / Guardrail)
- Any yes/no compliance flag in §0.3
- "Any more user stories?" (Yes — add another / No — move on)
- A diagram offer at §8, §9, or §11 (Yes, add it / No, skip)

Exceptions — keep as plain text:
- **Open-ended questions** (the working name in §0.1, evidence narrative in §3, hypothesis prose in §6, baseline/target values in §7): no discrete option set.
- **Questions with >5 effective options** (e.g., §0.2 initiative type with 8 options): keep as text, or split into two sequential button questions.
- **Multi-select** (e.g., §0.3 compliance signals — multiple may apply): use `AskUserQuestion` with `multiSelect: true`, up to 4 grouped categories.

If a question doesn't naturally fit any of these patterns, default to text — never invent options to force a button UI.

---

## Anti-Hallucination Rules

These rules apply at ALL times during Mode 1, Mode 2, and Mode 3. They are not optional.

### When the user does not have information

If the user says they don't know a detail — or if no answer is provided — use the most specific `[TBD]` variant defined in the rules below. If no specific variant applies, write `[TBD]`. Do NOT invent a plausible-sounding value.

This applies without exception to:
- Baselines and targets in §4 (Objectives / KRs) and §7 (Metrics)
- Event names and metric mappings in §11 (Tracking)
- Design artifact links in §9 (Solution)
- Stakeholder names in §2, §10, §16
- Evidence, sources, and citations in §3 (Background)
- Dates, deadlines, and review schedules in §10 and §15
- Risk owners, dependency confirmations, and compliance references
- Dashboard or monitoring tool names and links in §10

### Specific prohibitions

- NEVER write a metric baseline or target that the user did not provide. Write `[TBD]` instead.
- NEVER fabricate a design link, Figma URL, Confluence link, or Jira ticket. Write `[No design link — status: Draft]` instead.
- NEVER invent a person's name as a reviewer, approver, DRI, or owner. Write `[TBD — owner: ]` or `[TBD — approver: ]` instead.
- NEVER write "approximately", "roughly", or "around X%" to soften an invented number. If the user didn't give the number, write `[TBD]`.
- NEVER generate evidence, research citations, or data to support the background if the user has not provided it. Write unvalidated assertions as: `[Team belief — unvalidated: ___]`.
- NEVER fabricate an event name, tracking property, or data destination. If the user has not confirmed a tracking plan, mark every event row: `[TBD — event name to be confirmed by data team]`.
- NEVER pre-populate the data team sign-off checkbox as complete unless the user explicitly confirms it.
- NEVER describe a UI or design in prose if no design link exists. Use: `[Design pending — link to be added]`.

### When the user is vague

Push back with a clarifying probe before writing the section. Use the probes defined in `prompts/interview-questions.md`. Do not write the section until you have enough real information to avoid fabricating core content.

### [TBD] handling in validation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anatasof/NatPRD](https://github.com/anatasof/NatPRD) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
