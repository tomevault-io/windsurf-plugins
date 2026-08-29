---
trigger: always_on
description: **All AI outputs must be in English**, regardless of the language used in user prompts. This applies to code, comments, documentation, configuration files, commit messages, and response text.
---

## Language Policy

**All AI outputs must be in English**, regardless of the language used in user prompts. This applies to code, comments, documentation, configuration files, commit messages, and response text.

## Memory Policy

**Do not use Claude memory files to store project information**. All project knowledge — domain context, team structure, constraints, decisions, and any other relevant information — must be captured exclusively through the SDLC artifact system (stakeholders, constraints, assumptions, goals, requirements, decisions, etc.). This ensures all knowledge is structured, traceable, and available to every team member working on the project.

---

## Project Overview

<!-- Replace this section with a description of your project. -->

This repository uses a structured, AI-first development lifecycle. All project knowledge — specification, design, decisions, tasks — lives alongside the source code.

### Current State

**Phase**: Not initialized

**Summary**: Pristine framework, not yet initialized. The repository contains the AI SDLC Framework (phase directories, templates, automation skills), ready to be populated starting from the Specification phase after initialization (`/SDLC-init`).

---

## Current State Protocol

The `### Current State` subsection above is the machine-readable project status shared by all skills. Its structure:

- **`**Phase**:`** — mandatory first line. One of: `Not initialized`, `Specification`, `Design`, `Code`. Every skill validates its applicability against this field before acting.
- **`**Summary**:`** — mandatory. One or two sentences of free text describing where the project stands.
- **Standardized status lines** — each optional, appearing at most once, added when its activity first occurs, then updated in place; never removed:

| Line | Format | Primary maintainer |
|------|--------|--------------------|
| `**Spec artifacts**:` | artifact types worked on so far | SDLC-elicit |
| `**Gap analysis**:` | active or passed form — see Assessment lines below | SDLC-elicit |
| `**Design documents**:` | drafting status per document | SDLC-design |
| `**Completeness assessment**:` | active or passed form — see Assessment lines below | SDLC-design |
| `**Components**:` | comma-separated component names | SDLC-decompose |
| `**Implementation plan**:` | `created YYYY-MM-DD — N phases, M tasks` | SDLC-implementation-plan |
| `**Task progress**:` | `D/M tasks done — currently in Phase N (name)` | SDLC-execute-task |

**Assessment lines** — `**Gap analysis**:` and `**Completeness assessment**:` record the run date, a `fresh`/`stale (reason)` marker, and the compact list of open issues: `YYYY-MM-DD — fresh — open: <Severity>: <short summary>; …` (or `— no issues` when the run found none). An issue is removed from the list when a change corrects it; when the last one goes, the tail becomes `all issues resolved`. A new run overwrites the whole line. When the corresponding phase gate is passed, the issue list is removed, keeping the date and how the gate was passed: `passed: no issues`, `passed: all issues resolved`, or `passed: user accepted remaining issues (<counts by severity>)` — `not performed — passed: user accepted to proceed` if no run was recorded. A line with a `fresh`/`stale` marker is in **active form**; after the gate it is in **passed form**.

Maintenance rules — they apply to **every** artifact change, whether performed inside a skill or not:

1. **Same-operation update**: update the affected lines in the same operation as the artifact change they reflect.
2. **Staleness** (active-form assessment lines only): changes to Specification artifacts flip `**Gap analysis**:` to `stale (artifacts changed since)`; changes to design documents or decisions flip `**Completeness assessment**:` to `stale (design changed since)`, and Specification changes flip it too (`stale (spec changed since)`). Exception: a change that only corrects listed open issues updates the list instead, and status-only lifecycle advancements (e.g., `Approved → Implemented`) do not flip the marker.
3. **Phase transitions**: `**Phase**:` changes only when a phase gate is crossed (see Phase Gates); the crossed gate's assessment line switches to its passed form.

---

## Working Agreement

All changes to this repository are made by AI agents following these instructions — ideally through the skills, which encode the full procedures.

- **Framework vs. project content**: [`FRAMEWORK.md`](FRAMEWORK.md) declares which files belong to the framework and which to the project, with modification and upgrade rules per category — consult it before modifying anything beyond project content.
- **Free-prompting** (agent work outside a skill) carries risk: an agent acting in a context a skill covers, without following that skill's procedures (see Cross-Skill Artifact Procedures below), can introduce inconsistencies. Prefer invoking the skill; otherwise follow its procedures.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pangon/ai-sdlc-framework](https://github.com/pangon/ai-sdlc-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
