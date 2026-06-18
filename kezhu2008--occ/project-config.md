---
trigger: always_on
description: Use when standing up, running, or re-staffing an AI "company" that builds and maintains a specific code repository — turning a founder's business intent into a structured org of CEO / manager / executor / reviewer / HR persona skills scoped to that repo. Triggers include "spin up a team for this project", "run my company", "review the org", or wanting autonomous repo development with minimal founder intervention.
---


# OCC — One Consultancy Company

## Overview

OCC is the **staffing-and-consulting firm**. It does not build your product. It builds (and re-builds) the **org that builds your product**: a structured team of persona skills scoped to one code repository.

**A company = one code repository pursuing one business intent.**

OCC has exactly two capabilities — the same two a real consultancy sells:

1. **Strategic review** (`occ-strategic-review`) — interview the founder, design or revise the org: roles, reporting lines, job descriptions. Output is versioned org design in `company_policies/`.
2. **Recruiting** (`occ-recruiting`) — instantiate that design into real, invokable persona skills under the repo's `.claude/skills/`, each reusing superpowers and each paired with a counter-reviewer.

OCC makes **structural** change (new/removed/redefined roles). Inside the company, **HR** makes **incremental** change (training a persona from accumulated feedback). Two different gears — keep them separate. See `references/corporate-structure.md`.

## When to use

- "Set up a team to build/maintain `<repo>`" → run `occ-strategic-review`, then `occ-recruiting`.
- "The team is missing a role / a role is wrong" → `occ-strategic-review` (revise), then `occ-recruiting` (re-recruit just the delta).
- "Run my company / ship the next milestone" → you are past OCC; invoke the recruited **CEO** persona (`<company>-ceo`). OCC built it; the CEO runs it.
- Coaching one persona to behave better without changing the org → that's **HR** (`<company>-hr`), not OCC.

## The three-tier storage model (read this first)

Everything OCC and the company produce lands in one of three tiers. Confusing them is the most common failure.

| Tier | Lives in | Lifespan | Examples |
|------|----------|----------|----------|
| **Persistence** (authoritative facts) | `<repo>/company_policies/` | Whole project, git-tracked | charter, org-chart, roadmap, architecture, data-model, **specs (R#/A#)**, use-cases, tasks, decisions, performance |
| **Knowledge** (shared map + gotchas) | `<repo>/company_policies/knowledge/` | Until superseded | "FX routes through Div775Engine not CGT — see `Div775Engine.swift:40`; missing a USD movement silently under-reports tax" |
| **Memory** (short-term role feedback) | `<persona>/MEMORY.md` | Until HR triages | "CEO over-explained architecture", "iOS dev skipped device check" |

Knowledge is **shared** and kept small by construction — entries *reference* the authoritative artifact, never restate it; HR gardens it (dedup, supersede, budget). **HR** is the only function that promotes: behavior → SKILL.md (writing-skills, RED baseline), gotcha → knowledge/, fact → a spec. See `references/memory-vs-persistence.md`.

## Canonical repo layout OCC produces

```
<company_repo>/
  company_policies/            # PERSISTENCE — versioned, founder-facing
    charter.md                 # business VISION only (founder + CEO); no tech design
    org-chart.md               # the persona registry: role → skill → reviewer → reports-to
    job-descriptions/<role>.md # one per role; the recruiting spec
    roadmap.md                 # milestones + success criteria (CEO-owned)
    architecture.md            # current-system shape, seams, invariants (ARCHITECT-owned)
    data-model.md              # types, schema, persistence (ARCHITECT-owned)
    specs/<area>.md            # backend spec R#/A# (PM + ARCHITECT co-owned)
    system-design.md           # next-phase deployment, API, migration (TECH-LEAD-owned)
    use-cases/                 # step-by-step use-case catalog, backed by specs/ (PM-owned)
    ux/                        # UX manuals (UX-manager-owned)
    milestones/<id>.md         # per-milestone spec (CEO-owned)
    tasks/<id>.md              # per-task spec; acceptance = the spec's A# (tech-lead-owned)
    test-strategy.md           # acceptance/integration test plan (QA-owned)
    knowledge/                 # KNOWLEDGE tier — shared map + gotchas, HR-gardened
    decisions.md               # autonomous decision log + the outcome that harnessed each
    handbook.md                # handoff, review bar, consultation, escalation, automation
    performance/<date>.md      # HR reviews + failure-mode catalog + knowledge GC
  .claude/skills/              # the recruited org — invokable persona skills
    <company>-ceo/SKILL.md     +  <company>-board-reviewer/   # CEO's plans reviewed before founder
    <company>-product-manager/ +  <company>-pm-reviewer/
    <company>-ux-manager/      +  <company>-ux-reviewer/
    <company>-architect/       +  <company>-architect-reviewer/   # correctness (design)
    <company>-tech-lead/       +  <company>-tech-lead-reviewer/   # delivery (≠ architect)
    <company>-<executor>/      +  <company>-<executor>-reviewer/  # one reviewer per executor
    <company>-qa-engineer/SKILL.md                              # product QA (≠ the reviewers)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kezhu2008/occ](https://github.com/kezhu2008/occ) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
