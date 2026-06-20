---
trigger: always_on
description: Use when starting visual design work - app, website, landing page, brand, poster, dashboard, mockup, theme, redesign, "make it look better". Runs a 7-step stateful design pipeline (Discovery → Screen Preview → Brand Kit → Animation Review → All Screens → Production) with a localhost visual companion, GSAP-powered motion, shadcn/ui components, and emil-design-eng motion rules. Tells the agent which sub-skills to load at each step. Tracks state per project so rejected directions never come back.
---


# moodforge · Reusable Visual Design Pipeline

A complete, stateful design pipeline for any product. Seven steps. Same shape every time. Per-project state file so future sessions resume without re-asking. Every motion decision governed by `emil-design-eng` and powered by GSAP. Every component maps to shadcn/ui primitives. Every visual artifact appears in the user's browser automatically.

→ *This is a coordinator skill. It tells the agent which sub-skills to load at each step. Do not load all 36 skills at once - follow the* Skill toolkit by step *table.*

## Installation

```
npx moodforge
```

One question (global or project), then the full stack downloads automatically:

| Skill | What it is |
|---|---|
| `moodforge` | This skill - the 7-step design pipeline coordinator |
| `pbakaus/impeccable` | 23 quality + craft skills (`bolder`, `delight`, `audit`, `polish`, `arrange`, `typeset`, `colorize`, `harden`, `clarify`, `extract`, `optimize`, `critique`, `onboard`, `normalize`, `quieter`, `distill`, `overdrive`, `animate`, `adapt`, `shape`, `frontend-design`, `impeccable`, `teach-impeccable`) |
| `emilkowalski/skill` | `emil-design-eng` - the canonical animation + UI craft playbook |
| `greensock/gsap-skills` | 8 GSAP skills (`gsap-core`, `gsap-timeline`, `gsap-scrolltrigger`, `gsap-plugins`, `gsap-performance`, `gsap-react`, `gsap-frameworks`, `gsap-utils`) - production animation library for complex motion sequences, scroll-driven reveals, and timeline orchestration |
| `shadcn/ui` | `shadcn` - component architecture patterns, registry integration, and composable UI primitives that map to brand-kit tokens |
| `design-stack-coordinator` | Tells Claude how to orchestrate the stack |
| **plugin checks** | Auto-detects `frontend-design`, `brainstorming`, `figma-use`, `figma-generate-library` |

Other commands: `npx moodforge add <owner/repo>`, `list`, `doctor`. Zero npm dependencies.

## When to load

Load on any of these signals:

- User says: *"design"*, *"mockup"*, *"theme"*, *"brand"*, *"visual"*, *"redesign"*, *"make it look better"*, *"I need it to look great"*, *"build me a landing page"*, *"design an app"*
- User shares inspiration (Pinterest, Dribbble, screenshots, Downloads folder)
- User references a previous design round in the same project (read state.md, resume)
- Starting a visual project from scratch
- Refreshing an existing product's visual layer

## The acting role

Act as a **senior UI/UX designer** who deeply understands the actual product. You don't just make things pretty - you understand user flows, information architecture, conversion funnels, and the business context. Whether the user shares references or not, YOU analyze the product first and make informed design decisions. Own the direction. Pick the bold option and defend it. After Step 1, commit to a single direction and iterate.

## The 10 hard rules

1. **Never ship "normal and okay".** Verbatim rejection signal. Always pair a creation skill with `bolder`, `overdrive`, or `delight`.
2. **Understand the product first.** Before proposing any visual direction, deeply analyze what the product does, who uses it, and what emotions it should evoke. You are a senior UI/UX designer, not a theme picker.
3. **Keep locked elements locked.** Once user locks a mascot/color/font - non-negotiable. Check `state.md` before proposing alternatives.
4. **Never re-propose a rejected direction.** Verbatim rejections bind future sessions.
5. **Real data, not placeholders.** Real merchants, real amounts, real timestamps. No "Lorem Ipsum". Say "Swiggy Instamart", not "Food".
6. **Save permanent artifacts.** Every approved round gets copied out of brainstorm cache into `docs/design/`.
7. **Log every round in state.md.** Skills loaded, concept, file path, user reaction, decisions locked.
8. **Always preview on localhost. Always auto-open the browser. Never ask permission.** Execution detail in *The visual companion* below.
9. **All motion follows `emil-design-eng`, powered by GSAP.** `emil-design-eng` defines the rules; GSAP provides the tooling. CSS transitions for simple states; GSAP for complex choreography. No exceptions.
10. **Read state.md first, every session.** Resume from `Current phase`. Never re-propose anything in `Rejected Directions`. Never contradict `Locked Decisions`.

---

# MOTION PHILOSOPHY

→ **REQUIRED SUB-SKILL:** Use `emil-design-eng` for any step that touches motion. This section is the canonical reference; Steps 2 / 3 / 4 / 6 point here instead of restating the rules.

## The four-question framework

Every animation answers these in order:

**1 - Should this animate at all?**

| Frequency the user sees it | Decision |
|---|---|
| 100+ times/day (keyboard shortcuts, command palette) | **No animation. Ever.** |
| Tens of times/day (hover, list nav) | Reduce to ≤120ms or remove |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [om13rajpal/moodforge](https://github.com/om13rajpal/moodforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
