---
trigger: always_on
description: When the user types /plan or asks to create a project plan, feature PRD, or retrospective
---


# Plan — Structured Project Planning & PRD Generation

Three planning modes with a shared research-driven Q&A engine. Pattern across all modes: **research → questions → document → review → issues**.

## Phase 0: Read Session Config

Read Session Config from CLAUDE.md. Required field: `plan-baseline-path`. If missing or absent, stop with error:

> "Error: `plan-baseline-path` is not configured in Session Config. Add it to your CLAUDE.md under `## Session Config`. Example: `plan-baseline-path: ~/Projects/projects-baseline`"

Additional fields: `plan-default-visibility`, `plan-prd-location`, `plan-retro-location`, `vcs`. Expand `~` in paths.

## Phase 1: Mode Selection

Parse the argument to determine mode:

- **`new`** — Project kickoff: scaffolding, architecture decisions, initial setup
- **`feature`** — Feature PRD: requirements gathering, compact scope, acceptance criteria
- **`retro`** — Retrospective: data-driven analysis of recent work, learnings extraction

If no mode specified, ask via numbered list:

```
Which planning mode?

1. new (Recommended) — Project kickoff (full PRD, repo setup, issue creation)
2. feature — Feature PRD (compact scope, acceptance criteria, issues)
3. retro — Retrospective (metrics analysis, reflection, improvement actions)
```

## Phase 2: Q&A Engine (Shared Core)

Every question wave follows the same pattern. This is the core mechanic across all modes.

### 2.1 Pre-Question Research

Before each Q&A wave, perform research **sequentially** — Cursor has no parallel agents:

1. **Market/online context** — Search for relevant market data, best practices, competitor analysis, or technical patterns for the upcoming questions
2. **Baseline analysis** — Read projects-baseline templates, rules, and scripts at `$BASELINE_PATH` (Glob, Grep, Read)
3. **Repo context** — Analyze current repository for patterns, file structure, dependencies, conventions (skip for `/plan new` wave 1)

### 2.2 Question Presentation

Synthesize research into **5 questions per wave**. Present as numbered Markdown lists — Cursor has no AskUserQuestion:

- **Option 1 is ALWAYS the recommendation**, marked with `(Recommended)`
- Include Pros/Cons drawn from the research for each option
- Include an "Other" option when custom input makes sense

Example:
```
## Architecture (Wave 1, Q1)

Which project archetype fits best?

1. nextjs-saas (Recommended) — Pro: Full SaaS stack with auth, payments. Con: Heavier setup.
2. express-service — Pro: Lightweight API. Con: No frontend.
3. docker-service — Pro: Maximum flexibility. Con: More manual setup.
4. Other — Describe your preferred archetype.
```

### 2.3 Adaptive Depth

Starting wave counts: `new` → 3 waves minimum, `feature` → 1 wave minimum, `retro` → 1 wave minimum. Maximum 5 waves across all modes.

After each wave:
- **Answers clear** → proceed to document generation
- **Complexity revealed** → add targeted follow-up wave
- **User aborts** → proceed with answers gathered

### 2.4 Answer Tracking

After each wave, output a recap:
```
## Answers So Far (Wave N/M)
1. Archetype: nextjs-saas
2. Visibility: internal
3. Audience: B2B customers
```

---

## Mode: new — Project Kickoff

**Wave 1** — Core Decisions: archetype (read `$BASELINE_PATH/templates/`), visibility, target audience, core problem, GitLab group.

**Wave 2** — Technical Details (per chosen archetype): tech stack decisions, design style, external integrations, performance requirements, security requirements.

**Wave 3** — Business & Scope: MVP appetite (1w/2w/6w), success criteria (SMART), known risks, post-launch plan, ecosystem dependencies.

Document: 8-section full PRD. Save to `{plan-prd-location}/YYYY-MM-DD-{project-name}.md`.

After PRD approval: run `$BASELINE_PATH/scripts/setup-project.sh`, verify repo, populate CLAUDE.md, commit PRD.

Issues: Epic (project name) + sub-issues per MVP feature from PRD Section 4 (Solution & Scope).

---

## Mode: feature — Feature PRD

**Wave 1** — Feature Core: what to build, why now, who uses it, scope + explicit exclusions, dependencies.

**Wave 2** (conditional — only if Wave 1 reveals multiple subsystems or unclear integration): architecture decisions, integration points, data model changes, edge cases, performance impact.

Document: 5-section compact PRD. Save to `{plan-prd-location}/YYYY-MM-DD-{feature-slug}.md`.

Issues: Epic (feature name) + sub-issues per acceptance criterion group.

---

## Mode: retro — Retrospective

**Phase 1 (automatic, no user input):** Read `.orchestrator/metrics/sessions.jsonl`, run git log analysis, query open issues, compare last 5 vs prior 5 sessions.

**Wave 1** — What went well / what didn't: highlights, blockers with root causes, carryover items, process assessment, data anomalies.

**Wave 2** (conditional — if Wave 1 reveals significant blockers or process issues): improvement actions, priority ranking, ownership, baseline changes, Session Config adjustments.

Artifacts: retro doc → `{plan-retro-location}/YYYY-MM-DD-retro.md`. Improvement issues + learnings update (`.orchestrator/metrics/learnings.jsonl`).

---

## Phase 3: Document Generation

Read the appropriate template from the skill directory:
- `new` → `prd-full-template.md` (8 sections)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kanevry/session-orchestrator](https://github.com/Kanevry/session-orchestrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
