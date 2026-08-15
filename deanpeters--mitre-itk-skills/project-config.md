---
trigger: always_on
description: The primary audience is **product practitioners**: product managers, product owners, product builders, product leaders, and business analysts. These are people who:
---

# CLAUDE.md — Working in the MITRE ITK Skills Repo

## Who This Repo Serves

The primary audience is **product practitioners**: product managers, product owners, product builders, product leaders, and business analysts. These are people who:

- Write PRDs, product strategies, and roadmaps
- Run discovery sprints and user research
- Manage stakeholders and facilitate cross-functional alignment
- Work in agile environments (sprints, retrospectives, backlog refinement)
- Are familiar with Lean Startup, Jobs-to-be-Done, OKRs, and design thinking terminology

This is **not** a general facilitation library. When helping with this repo, frame everything in terms of product work.

---

## How Skills Are Organized

27 tools across 5 ITK phases:

- **SCOPE** (7 tools) — stakeholder mapping, system context, cultural framing
- **DEFINE** (3 tools) — problem framing, premortems, mission/vision
- **UNDERSTAND** (7 tools) — user research synthesis, journey mapping, value mapping
- **GENERATE** (4 tools) — structured brainstorming and ideation
- **EVALUATE** (6 tools) — testing, prioritization, retrospection

Each tool lives at `skills/itk-<alias>/SKILL.md` with assets in `skills/itk-<alias>/assets/`.

**Critical:** the `name:` field in every SKILL.md frontmatter must exactly match the directory alias (e.g. `name: itk-premortem`). This is what Claude Code reads to register the slash command. A mismatch causes the skill to load under the wrong name.

---

## SKILL.md Format

```
---
name: itk-<alias>
description: one-line description
intent: purpose statement
type: component
phase: scope | define | understand | generate | evaluate
outcome: phase name
difficulty: beginner | intermediate | advanced
group_size: e.g. "4+ people"
time_required: e.g. "45+ minutes"
best_for:
  - "specific PM scenario"
sources:
  - MITRE Innovation Toolkit (ITK)
  - "url"
---

# Tool Name

## What Is It
## Why Use It
## When to Use It
## How to Do It
## Key Concepts
## PM Applications
## Benefits
## Common Pitfalls
[## Combine With]
## Assets
## Metadata
```

---

## Recommending Tools

### Adaptive Decision Ladder

The **Adaptive Decision Ladder (ADL)** is a multi-step guided flow that narrows tool recommendations through a sequence of numbered choices. It serves two purposes: **performative** (it helps produce a better-fit recommendation by surfacing context the user may not have thought to provide) and **pedagogic** (it teaches practitioners how to think about which class of tool applies when).

**The ADL is a mode, not a gate.** Experienced users can skip it entirely — or exit it mid-flow — by:
- Naming a tool directly ("walk me through Premortem")
- Providing enough context to resolve the recommendation in one step ("I'm running a discovery kick-off for a new B2B feature, team of 6, 90 minutes")
- Saying "just recommend something" at any point

**When to offer the ADL vs. answer directly:**

| User input | Response |
|------------|----------|
| Names a specific tool | Go directly — explain, contextualize, or walk through it |
| Describes a clear situation with enough detail | Recommend directly; optionally note: *"If you'd like me to ask a few questions to sharpen this further, just say so"* |
| Asks a vague question ("which tool should I use?") | Start the ADL at Step 1 |
| Describes a situation that spans multiple phases | Start the ADL — use their description to pre-answer or skip steps that are already resolved |

**Read context before step 1.** Before presenting any options, check: What phase of work is already evident? What tools have already been mentioned? What artifacts (PRD, roadmap, OKRs) have been referenced? Skip or pre-answer any step that context already resolves — never ask what you already know.

**At any step:** if the user says "just recommend" or provides enough detail to jump ahead, skip remaining steps and go directly to Step 4.

**How it works:**
- Present a small numbered list at each decision point
- The user can reply with a single number (`2`), multiple numbers (`1 & 3`), or choose the open option to describe their situation in their own words
- Carry all prior selections and context forward into each subsequent step
- End with 1–3 specific tool recommendations, ranked, with brief rationale tied to their stated situation

---

### ADL Step 1 — Situation

Present this first:

> **What best describes where you are right now?**
> 1. Scoping — figuring out who's involved, what the system looks like, or what our culture needs
> 2. Defining — not sure what problem to solve, or need to pressure-test a direction
> 3. Understanding users — need research, journey maps, or clearer user needs
> 4. Generating solutions — have a defined problem and need ideas
> 5. Evaluating options — need to test, cut, prioritize, or assess tradeoffs
> 6. Reflecting — sprint retro, project wrap-up, or team health check
> 7. Other (describe)

---

### ADL Step 2 — Narrow by specific need

Branch on their Step 1 answer. Present the relevant sub-menu:

**If 1 (Scoping):**
> 1. I don't know who all my stakeholders are yet
> 2. I know my stakeholders but need to prioritize who to engage

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [deanpeters/MITRE-ITK-Skills](https://github.com/deanpeters/MITRE-ITK-Skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
