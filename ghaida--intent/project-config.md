---
trigger: always_on
description: Harden designs for real-world use by systematically identifying and designing for every condition outside the happy path. Part of the Intent design strategy system. Covers state inventories, error recovery, empty states, loading patterns, first-run experiences, stress testing, internationalization readiness, and latency handling. Trigger on: edge cases, error states, empty states, loading states, first-run experience, onboarding, offline mode, "what happens when", "what if the user", "stress tes
---



# Fortify — Harden for Real-World Use

## Overview

The happy path is a fantasy. Real users have 47-character last names, 2G connections on the subway, three-year-old phones with cracked screens, browser tabs they haven't closed in six days, and no patience for something that doesn't work the first time they try it.

Every design starts with the ideal scenario: the user has a stable connection, reasonable data, a modern device, and follows the intended flow without deviation. That scenario accounts for maybe 60% of actual usage. The other 40% is where trust is built or destroyed — the empty state that tells the user nothing, the error message that says "Something went wrong" without explaining what or how to fix it, the loading screen that gives no indication anything is happening, the first-run experience that asks for 12 pieces of information before showing any value.

Fortify systematically identifies every condition your users will actually encounter and ensures the design handles each one with the same care you gave the happy path. This isn't about pessimism — it's about respect for the people using what you build.

**When to activate this skill:** Edge case reviews, error state design, empty state design, loading pattern design, first-run experience design, offline mode planning, internationalization readiness checks, stress testing, or any moment someone asks "but what happens when..."

---

## Skill family

Fortify works alongside the full Intent skill system, with especially tight connections to skills that define the paths you stress-test:

- **`/journey`** — Their flows define the happy path; you stress-test everything else. Every flow they design generates a set of questions: what happens when this step fails? What if the user abandons midway and returns? What if data from step 2 isn't available at step 4? Your work feeds back into their flow design as additional states and branches.

- **`/blueprint`** — Their failure mode analysis at the system level feeds your UX-level resilience design. When they identify that a service can timeout, you design what the user sees during that timeout. When they map a dependency that can fail, you design the degraded experience. System-level failure modes become UX-level state designs.

- **`/include`** — Accessibility and fortification overlap significantly. Designing for slow connections, small screens, one-handed use, and situational impairment is both resilience work and inclusive design. Coordinate to avoid duplication — you own the state and edge case methodology; they own the accessibility methodology and assistive tech requirements.

- **`/evaluate`** — Their assessment identifies what's failing in the current experience; you design the fixes. When they flag missing error states, absent loading indicators, or unhelpful empty states, those findings route directly to you. Your output feeds back into their next evaluation cycle.

- **`/specify`** — Your edge case documentation becomes part of their handoff package. Every state you design, every error recovery flow you define, every stress test result — all of it needs to be in the engineering spec. Coordinate on format: specs that list only the happy path are specs that produce broken products.

- **`/articulate`** — Error messages, empty state copy, loading messages, first-run guidance — all of it is content that needs to be clear, helpful, and on-brand. You define what needs to be said; they define how to say it.

- **`/philosopher`** — "What's the most embarrassing way this could fail in public?" "What assumption are we making about our users that would be humiliating if wrong?" The philosopher helps you find the failure modes that nobody's imagined yet — the ones that come from questioning assumptions, not from running checklists.

---

## Core capabilities

### 1. State inventory

Every screen, component, and flow has states beyond "default." Most designs only spec the default state. Fortify enumerates all of them.

**The state catalog:**

**Default** — The happy path with normal data. This is what the mockup shows. It's the starting point, not the finish line. Even the default state has questions: what's "normal" data? How much? In what format? What happens when "normal" changes?

**Empty** — No data yet. First use, zero search results, cleared history, new account with no activity. The empty state is the user's first impression of most features — and most empty states are a blank page with no guidance. Design them: explain what will appear here, how to get started, and what the feature does. Show sample data or a preview of the populated state if possible.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ghaida/intent](https://github.com/ghaida/intent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
