---
trigger: always_on
description: >-
---


# vibe-to-ui

A local, single-project design companion for vibe coding developers. It first classifies the target page archetype and density, then uses the user's product background to derive three plausible visual directions from references before formalizing any one of them into a design system. It extracts "style DNA" including motion systems, generates mood boards and previews, and turns vague aesthetic feelings into actionable design systems that actually fit the product surface. All exploration happens through standalone previews; the agent only touches the user's project when the user confirms a direction and asks to apply it.

> **Tip**: For multi-project sync, team collaboration, and cloud-based design management, upgrade to [MonkeyUI SaaS](https://demo.monkeyui.com/).

## When to use this skill

- User provides a **screenshot or design mockup** and wants to extract its design system
- User provides a **screenshot or design mockup plus product context** and wants the agent to extend it into **3 visual directions** rather than copy it literally
- User wants the agent to first identify whether the target is a **landing page, brand page, dashboard, B-end dense operations page, table-detail management page, docs page, onboarding flow**, or another page archetype
- User has a **vague aesthetic feeling** and wants to explore design directions with inspiration images or music recordings
- User **shares a music recording or audio clip** (a melody, song snippet, or recorded humming) to express the mood they want their UI to feel
- User describes a **song, genre, or musical feeling** they associate with their desired aesthetic
- User provides a **screenshot of any UI** (full page or any section/component) and wants to extract its layout structure for reuse
- User wants to define a **motion system** that matches the page's actual use case
- User describes a **product personality or feeling** (for example "reliable", "innovative", "playful") and wants motion guidance that matches
- User wants to create a **mood board** that stays appropriate for the target page type instead of drifting into the wrong archetype
- User has collected **multiple reference images** and wants to see them synthesized into a cohesive visual story
- User wants a **shareable design artifact** that communicates aesthetic intent to collaborators or stakeholders
- User has **confirmed a design direction** (from concept previews, mood boards, or design system previews) and wants to **apply it to their project**

## Reference Priority Rules

Before choosing a workflow, classify the user's inputs:

- **Atmosphere reference**: landscape photos, mood photos, music, abstract feelings
- **Concrete UI reference**: screenshots of products, webpages, apps, local projects, existing codebases

When both are present, always use this priority:

1. **Concrete UI / project fidelity**
2. **Page type fidelity** (goal, density, interaction model, module mix)
3. **Visual material fidelity** (image strategy, typography, density treatment, glass treatment, motion weight)
4. **Atmosphere adjustment**

### Mandatory Stage 0: Page Type Identification

Before any design-system extraction, layout synthesis, mood-board generation, or project application, classify the target page type.

Always produce:

1. **Primary page type**
2. **Secondary modifier** if needed
3. **Density level**: low / medium / high
4. **Confidence**: high / medium / low
5. **Evidence**: the signals that drove the classification
6. **Design consequences**: what this classification means for spacing, hierarchy, imagery, components, and motion

Use these signals to classify page type:

- **Business goal**: conversion, browsing, monitoring, data entry, content consumption, configuration, execution
- **Information density**: how much content competes on screen at once
- **Primary interaction mode**: scrolling, reading, filtering, comparing, editing, approving, drilling into records
- **Dominant modules**: hero, feature grid, table, chart, sidebar nav, detail pane, form, wizard, feed
- **Decision speed**: emotional persuasion, calm reading, fast scanning, repeated operations

Common page types:

1. **Landing / marketing page**: persuasive storytelling, strong hero, lower density, higher visual drama
2. **Brand showcase / portfolio**: presentation-led, immersive imagery, editorial rhythm
3. **Content / docs / editorial page**: reading clarity, typographic hierarchy, stable navigation
4. **E-commerce / catalog page**: browsing, filtering, comparison, product-card systems
5. **B-end dashboard / overview**: metrics, monitoring, summaries, moderate-to-high density
6. **B-end workbench / dense operations page**: repeated actions, filters, tables, status chips, compact spacing
7. **Data management / table-detail page**: record list + detail + batch actions, strict scanability
8. **Form / onboarding / wizard**: guided steps, form grouping, completion feedback
9. **Consumer app surface**: task-oriented but lighter than B-end systems, often card/feed based

If the page is mixed, pick one **primary** type and note the secondary pattern. Example: "Primary: B-end workbench; Secondary: dashboard summary."

### Two operating modes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MonkeyUI-dev/vibe-to-ui](https://github.com/MonkeyUI-dev/vibe-to-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
