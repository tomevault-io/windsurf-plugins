---
trigger: always_on
description: Structured software design before coding. Use when the user wants to build a new project, app, or feature from scratch — including 'I want to build...', 'spec vibe', 'help me plan this project', 'I need a PRD', or any new coding project request. Also triggers when the user jumps straight to 'write me an app that does X' — redirect them to design first.
---


# Spec Vibe Coding — Vibe coding with specs, not chaos

## What this skill does

Spec Vibe Coding guides users through a professional software design process before they write a single line of code. It produces a complete set of design documents — PRD, tech spec, UI spec, and development task breakdown — saved as markdown files in `.spec-vibe/` in the project directory.

The core insight: AI coding tools are great at writing code, but they need good instructions. Most projects fail not because the code is bad, but because nobody thought through what to build, how to organize it, or what the user experience should be. Spec Vibe Coding fixes this by making "thinking before coding" structured, guided, and even enjoyable.

## Who this is for

People who have an idea and want to build software — whether they're non-technical founders, junior developers, indie hackers, or experienced engineers who tend to skip the design phase. The skill adapts its language and depth based on the user's technical level.

## How it works

Spec Vibe Coding runs as a multi-phase conversation. Each phase produces a markdown document saved to `.spec-vibe/`. The user can go through phases sequentially or jump between them. At any point they can say "let's start coding" to generate development tasks from whatever design work is complete.

---

## Phase overview

```
Phase 0: Project kickoff       → .spec-vibe/00-blueprint.md
Phase 1: Requirements           → .spec-vibe/01-requirements.md
Phase 2: System design          → .spec-vibe/02-system-design.md
Phase 3: UI/UX design           → .spec-vibe/03-ui-design.md
Phase 4: Development tasks      → .spec-vibe/04-dev-tasks.md
Phase 5: Quality check          → .spec-vibe/05-quality-report.md
```

Always create the `.spec-vibe/` directory at the start. If it already exists, read existing documents to understand current state and pick up where the user left off.

---

## Phase 0: Project kickoff

**Goal:** Turn a vague idea into a structured project blueprint.

**How to run this phase:**

1. Ask the user to describe their idea in their own words. Accept any format — a sentence, a paragraph, bullet points, or even "something like X but for Y."

2. Ask 2-4 targeted follow-up questions to fill gaps. Pick from these based on what's missing:
   - Who is this for? (target users)
   - What's the core problem it solves?
   - What platform? (web, desktop, mobile, CLI)
   - What's the scale? (personal tool, startup product, enterprise)
   - Any technology preferences or constraints?
   - What's the timeline expectation?

   Present questions with suggested options the user can pick from, plus room for custom answers. Don't ask more than 2-3 questions at a time. The user can say "that's enough" at any point.

3. Generate a project blueprint and save it.

**Blueprint document structure** (save to `.spec-vibe/00-blueprint.md`):

```markdown
# Project blueprint: {Project name}

## One-line description
{What this software does, in one sentence}

## Problem statement
{What problem does it solve, and for whom}

## Target users
{2-3 user types with brief descriptions}

## Core features (initial)
{Bulleted list of 5-10 key features, ranked by importance}

## Technical direction
{Platform, suggested tech stack, key constraints}

## Estimated complexity
{Simple / Medium / Complex — with brief justification}

## Next steps
{Which design phases to focus on}
```

4. Show the blueprint to the user. Ask: "Does this capture your vision? Anything to change before we dive deeper?"

5. After confirmation, tell the user which phase to tackle next (usually Phase 1) and that they can say the phase name anytime to jump there.

---

## Phase 1: Requirements (PRD)

**Goal:** Produce a complete product requirements document.

**Read `.spec-vibe/00-blueprint.md` first** to load project context.

Work through these 6 sections by asking the user focused questions. For each section, propose an AI-generated draft based on the blueprint, then let the user refine it. Don't dump all 6 sections at once — go one at a time.

### Sections

1. **Product positioning** — One-liner, core problem, competitive difference
2. **User personas** — Generate 2-3 persona cards (role, scenario, pain point, tech level). Ask the user to confirm, add, or remove.
3. **Feature list** — Organize into Core (MVP must-have), Enhanced (nice-to-have), Future (later). Each feature needs: name, description, and acceptance criteria ("how do we know it's done?"). This is where most users need the most help — prompt them on acceptance criteria specifically.
4. **Core user flows** — Extract 2-3 key journeys. For each: trigger → steps → outcome. Proactively ask about error cases and edge scenarios.
5. **Constraints and boundaries** — Tech constraints, time constraints, what's explicitly NOT in scope, MVP boundary definition.
6. **Success metrics** — How will the user measure if this project succeeded?

**Save to `.spec-vibe/01-requirements.md`** using this structure:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CheneyNine/Spec-Vibe-Coding](https://github.com/CheneyNine/Spec-Vibe-Coding) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
