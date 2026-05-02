---
trigger: always_on
description: Bridges design and engineering by producing detailed specs, organized handoff packages, asset inventories, and cross-functional documentation. Part of the Intent design strategy system. Trigger when: writing design specs, preparing engineering handoffs, documenting for development, creating design reviews, writing test plans, building copy matrices, addressing edge cases, aligning stakeholders, packaging designs "for engineering," or saying "write the spec," "prepare the handoff," "document this
---



# Specify — Bridge Design to Engineering

## Overview

This skill transforms design work into actionable, implementation-ready documentation. It produces structured specs, asset packages, test plans, and stakeholder presentations that ensure design intent survives to production. Use this when design needs to move into engineering, when cross-functional clarity is required, or when you must document decisions in a way that prevents rework.

---

## Skill family

Specify works alongside the full Intent skill system:

- **`/strategize`**: Their briefs and hypotheses provide the "why" behind everything you specify. Every spec should trace back to a strategic intent — why this feature exists, what hypothesis it tests, what user need it serves.
- **`/investigate`**: Their research findings ground your use cases in evidence. Real user quotes, observed behaviors, and validated pain points make specs persuasive and accurate, not hypothetical.
- **`/blueprint`**: Their system architecture constrains and informs your specs. Service dependencies, data flows, and technical constraints shape what's possible and what needs engineering discussion.
- **`/journey`**: Their flows are what you're specifying — screen sequences, interaction transitions, state changes. Journey designs the experience; specify documents it for implementation.
- **`/organize`**: Their information architecture informs your navigation specs. Taxonomy, hierarchy, and labeling decisions from organize become the structural backbone of your screen specs.
- **`/articulate`**: Their copy work feeds directly into your copy matrices. Voice, tone, and content strategy decisions become specific strings in your spec.
- **`/fortify`**: Their edge case analysis becomes part of your spec. Error states, failure modes, boundary conditions, and recovery patterns — all documented screen by screen.
- **`/include`**: Their accessibility requirements go into every screen spec. ARIA labels, keyboard navigation, color contrast, screen reader behavior — inclusion is not an appendix, it's woven into every screen.
- **`/evaluate`**: Their assessment identifies gaps in your specs. Heuristic violations, usability issues, and anti-pattern flags become items to resolve before handoff.
- **`/measure`**: Their success metrics define your test plan criteria. Every feature spec should include what success looks like, how to measure it, and what to instrument.
- **`/philosopher`**: A cross-cutting cognitive mode for when specification reveals deeper problems. Invoke when: edge cases keep multiplying, something about the design feels fragile under real conditions, the "pending questions" section keeps growing, or the user says "sit with this", "brainstorm", or "what could go wrong that nobody has imagined?" The philosopher helps think through failure scenarios nobody has considered and whether the spec is documenting the right thing.

---

## Core capabilities

### 1. Detailed design specifications

Write comprehensive, screen-by-screen (or state-by-state) specifications that document:
- Visual design with specific measurements, colors, typography, spacing
- Interaction logic: what triggers what, in what order, with what conditions
- Copy: exact text, variants for different contexts/markets/edge cases
- States: default, hover, active, error, loading, empty, success — all documented visually and logically
- Constraints: device sizes, performance requirements, accessibility needs

Output should be a living spec document (HTML or markdown) that engineers can reference during implementation without guessing.

### 2. Organized engineering handoff packages

Structure deliverables so engineering knows exactly what to build and why:
- Clear ownership: who decided what and when
- Problem context: what user need or business problem does this solve
- Design approach: constraints considered, alternatives rejected and why
- Use cases: specific, not generic — real user scenarios that expose edge cases
- Assets: all files organized, named, versioned, with usage notes
- Test criteria: success metrics and audience-specific test plans

### 3. Copy and variant matrices

Document all copy variations in one place:
- Primary copy vs. secondary copy vs. microcopy (labels, hints, errors, empty states)
- Market variants: tone shifts, cultural considerations, regulatory language
- Edge cases: character limits, long strings, very short strings, numeric edge cases
- A/B test variations: explicit copy changes being tested, with success criteria

### 4. Interactive HTML specification documentation

When appropriate, produce interactive HTML specs that:
- Show designs inline with explanatory text
- Link related screens and decisions
- Include collapsible reference sections (component specs, copy matrices, test plans)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ghaida/intent](https://github.com/ghaida/intent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
