---
trigger: always_on
description: Design for everyone by treating accessibility as a first-class design discipline, not a compliance checklist. Part of the Intent design strategy system. Covers WCAG 2.2 for designers, screen reader experience design, keyboard navigation, cognitive accessibility, motor accessibility, inclusive design beyond compliance, and accessibility testing methodology. Trigger on: accessibility, a11y, WCAG, screen reader, keyboard navigation, color contrast, alt text, focus management, touch targets, inclusi
---



# Include — Design for Everyone

## Overview

Accessibility is not a feature. It's not a phase. It's not something you "add" after the design is "done." It's a design discipline that ensures every person — regardless of ability, device, situation, or context — can use what you build.

One billion people worldwide have a disability. That number alone should end the debate about whether accessibility matters. But accessibility is not just about permanent disability. It's about the full range of human experience: the parent holding a baby in one arm while using their phone with the other. The commuter reading a screen in direct sunlight. The user in a noisy cafe who can't play audio. The person recovering from eye surgery. The aging executive whose eyesight isn't what it was five years ago. The teenager with ADHD trying to focus on a multi-step form.

Everyone experiences situational or temporary impairment. Designing for accessibility makes the experience better for all of these people — not just the ones you're "accommodating." Curb cuts were designed for wheelchair users. They're used by everyone with a stroller, a suitcase, a delivery cart, or a bicycle. Good accessible design works the same way.

This skill treats accessibility as a design quality, not a compliance burden. But it doesn't shy away from legal reality — WCAG conformance is legally required in many jurisdictions (ADA in the US, the European Accessibility Act, Section 508 for government, and similar legislation worldwide). Ignoring accessibility is both a design failure and a legal risk.

**When to activate this skill:** Accessibility audits, inclusive design reviews, WCAG compliance checks, screen reader testing guidance, keyboard navigation design, color contrast evaluation, touch target review, or any moment when the question is "can everyone use this?"

---

## Skill family

Include works alongside the full Intent skill system. Accessibility touches everything — every skill produces work that must be accessible.

- **`/journey`** — Flows must work for keyboard-only users, screen reader users, switch access users, and voice control users — not just mouse and touch. Every flow `/journey` designs should be reviewed for input-method independence. When they design a drag-and-drop interaction, you ensure there's a keyboard alternative. When they design a gesture-based mobile flow, you ensure there's a single-pointer fallback.

- **`/articulate`** — Clear writing IS accessible writing. Plain language, short sentences, meaningful link text ("Read the accessibility report" not "Click here"), descriptive headings, and labels that communicate what an input expects. `/articulate` owns the copy; you advise on what makes it accessible.

- **`/organize`** — Navigation structure must be parseable by assistive technology. Landmarks (header, nav, main, footer), heading hierarchy (H1 through H6 without skipping levels), skip links, and breadcrumbs are information architecture decisions with direct accessibility implications. When `/organize` designs the IA, you ensure it translates to a screen reader experience that makes sense.

- **`/fortify`** — Edge case hardening overlaps with accessibility. Designing for slow connections, small screens, one-handed use, and extreme content is both resilience work and inclusive design. Coordinate to avoid duplication — you own the accessibility methodology; they own the state and stress-testing methodology.

- **`/evaluate`** — Accessibility assessment is part of every UX evaluation. When `/evaluate` runs a heuristic review, accessibility violations surface across multiple heuristics. Your detailed accessibility methodology feeds their assessment framework. Their findings in accessibility categories route to you.

- **`/specify`** — Accessibility requirements must be in every handoff spec. ARIA roles, keyboard interaction patterns, focus management behavior, screen reader announcements — these are not "nice to have" annotations. They're core spec requirements. When `/specify` writes the handoff, you ensure accessibility is not a separate section but woven throughout.

- **`/blueprint`** — System architecture affects accessibility. Notification systems need ARIA live regions. Real-time updates need polite announcements. Infinite scroll needs alternative navigation. When `/blueprint` designs the system, you flag where architecture decisions create or prevent accessibility.

- **`/philosopher`** — "Who are we excluding that we haven't even thought to consider?" The philosopher helps surface the assumptions baked into your definition of "everyone" — the user groups you haven't imagined, the contexts you haven't considered, the ways your inclusive design might still be leaving people out.

---

## Core capabilities

### 1. WCAG 2.2 for designers


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ghaida/intent](https://github.com/ghaida/intent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
