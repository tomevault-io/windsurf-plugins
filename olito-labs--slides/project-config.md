---
trigger: always_on
description: Create professional HTML presentation decks using narrative-first methodology (dot-dash storylines, McKinsey/BCG pyramid structure, MECE assertion-evidence) with PDF export via Puppeteer. Three phases — (1) build ghost deck / dot-dash outline, (2) generate 1920x1080 pixel-perfect HTML slides with warm light-mode design system (Cormorant Garamond + Outfit, gold accent palette), (3) export combined PDF. Use this skill — NOT pptx — whenever building visually designed HTML slides. Triggers on "dot-d
---


# Slides — Narrative-First Presentation Skill

Three-phase workflow: **Narrative Architecture** → **Slide Creation** → **PDF Export**.

---

## Phase 1: Narrative Architecture (Dot-Dash)

Before opening a single HTML file, build the storyline. A deck is an argument, not a slideshow.

### 1.1 Identify the Governing Thought

Every deck answers one question. The governing thought is your **answer** — a single assertion, not a topic. If you can't write it in one sentence, you don't have a deck yet.

> **Answer-first principle:** State your conclusion before your evidence. The governing thought appears on the cover slide. Every headline states the answer, never the question. Business audiences want the punchline immediately.

The governing thought sits atop a **pyramid** (Minto Pyramid Principle):
- **Vertical logic:** Each level answers "Why?" or "How?" from the level above
- **Horizontal logic:** Arguments at the same level relate by deduction or induction

Key line arguments become slide headlines. Supporting data becomes slide bodies.

### 1.2 Build the Ghost Deck (Dot-Dash Outline)

Write one **headline** per slide (the "dot") and 2-4 **supporting points** (the "dashes"). Headlines are assertions, never labels.

**Bad:** "Market Overview"  →  **Good:** "The market is shifting from compliance-driven to intelligence-driven"

**MECE check:** Every level is Mutually Exclusive (no overlap) and Collectively Exhaustive (no gaps). Test: "Do any overlap?" (merge). "What's missing?" (add).

### 1.3 The Headline Test

Read headlines aloud in sequence as a paragraph. They should tell a coherent story without supporting detail. If the narrative breaks, restructure. Then apply the "So What?" test per-slide — if you can't connect it to the governing thought in one sentence, cut the slide.

### 1.4 Narrative Arcs

| Arc | Pattern | Best For |
|-----|---------|----------|
| **SCR** | Situation → Complication → Resolution | Pitches, proposals, enrollment |
| **SCQA** | Situation → Complication → Question → Answer | Board papers, decision memos |
| **What-So What-Now What** | Finding → Implication → Action | Recap decks, session summaries |
| **Before/During/After** | Past → Transformation → Current | Progress reports, case studies |
| **Problem-Mechanism-Proof** | Pain → How we solve it → Evidence | Product decks, demos |

### 1.5 Slide Type Assignment

Tag each slide in your dot-dash with its visual type. This determines layout in Phase 2. Choose the type that best serves the content — don't force content into a template.

| Type | Use For | Key Elements |
|------|---------|-------------|
| `cover` | Slide 1 always | Hero title, subtitle, badge |
| `skills-showcase` | Workflow demos, before/after | Two-column: narrative left, visual right |
| `constellation` | Architecture, team structures, networks | SVG diagram: nodes + hub + connectors |
| `timeline` | Journeys, stage progressions, roadmaps | Horizontal line with alternating above/below cards |
| `feature-grid` | Program components, offerings | 2×2 icon grid + philosophy/tagline bar |
| `team` | Coaches, founders, leadership | Circular photo crops + bios, vertical divider |
| `proof-asymmetric` | Track record, case studies, social proof | Asymmetric columns (~38/62), panel headings + rules |
| `investment` | Pricing, what's included, CTA | Items list + vertical divider + price card |
| `comparison` | Gap analysis, before/after | Two equal columns with card stacks |
| `metrics` | Impact, KPIs, results | Hero numbers with context labels |
| `pipeline` | Workflows, processes | Horizontal stage cards with connectors |

See `examples/` for reference implementations of each type. Copy the closest archetype, then customize — don't reproduce examples exactly. Each deck should feel designed for its specific audience.

---

## Phase 2: Slide Creation

### 2.0 The Consistency Canon

This is the most important section in the entire skill. When you flip through a professional deck, headers, titles, and footers don't move between slides. This creates a feeling of precision and polish that audiences register subconsciously. Inconsistency — even by a few pixels — looks amateurish.

**Every content slide MUST use these exact values:**

```css
/* HEADER — identical on every slide */
.header {
  display: flex; justify-content: space-between; align-items: center;
  padding: 48px 56px 0;
  position: relative; z-index: 10;
}
.logo-cluster { display: flex; align-items: center; gap: 16px; }
.logo-icon { height: 44px; width: auto; }
.logo-text {
  font-family: var(--font-body); font-size: 18px; font-weight: 600;
  letter-spacing: 0.18em; text-transform: uppercase; color: var(--text-muted);
}
.slide-label {
  font-family: var(--font-body); font-size: 14px; font-weight: 500;
  letter-spacing: 0.15em; text-transform: uppercase; color: var(--text-muted);
}


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Olito-Labs/slides](https://github.com/Olito-Labs/slides) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
