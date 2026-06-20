---
trigger: always_on
description: >-
---


# Markdown → Styled HTML

Convert a Markdown source file into a single self-contained HTML file that renders the *same content* as a finished web page — every word preserved, presented with deliberate typography, layout, and visual identity.

## The two invariants

These rules together define the skill. Everything else serves them.

1. **Text fidelity** — every word, sentence, list item, code snippet, link, table cell, and blockquote in the source Markdown appears unchanged in the HTML. No paraphrasing, no shortening, no "improving" the prose, no translation, no fixing what looks like a typo. The author's voice, including its imperfections, is sacred.

2. **Visual elevation** — the HTML is not a vanilla `<h1>` + `<p>` render. It uses considered typography, a deliberate color system, layout structure (sections, cards, grids where appropriate), and tasteful visual enhancements chosen to match the content's tone.

If these conflict, fidelity wins. Style supports the content; it never overrides it.

A useful self-check at the end: if someone copy-pasted the rendered text out of the HTML and diffed it against the source MD's text, the diff should be empty (modulo whitespace).

## Workflow

### 1. Read the whole source first

Read the entire MD file before writing any HTML. Picking a style without the full picture leads to a tone mismatch halfway down the page.

### 2. Diagnose the content's tone

Silently answer (don't write this in the output):
- **What is this?** Technical README, personal essay, product launch, course notes, recipe, novel chapter, internal memo, …
- **Who's the likely reader?** Developer, layperson, executive, friend, classmate, …
- **What emotional register?** Precise/technical, warm/personal, urgent/promotional, contemplative/literary, playful, somber, …

### 3. Commit to a single aesthetic direction

**Core principle — identify the medium the content imitates, and let that drive the aesthetic.**

- A **literary essay** imitates a **book page** → paper texture, serif body, ornamental dividers, narrow column.
- A **technical doc / status report** imitates a **documentation site** (Stripe Docs, Vercel Docs, Anthropic Docs) → screen-native, crisp hierarchy, clean canvas — *not* paper, *not* serif Latin body.
- A **product / startup launch** imitates a **modern product landing page** (Linear, Resend, Plain) → confident, designed, screen-native, bold typographic choices.
- A **recipe** imitates a **magazine spread**. A **course note** imitates a **notebook**. A **poem** imitates a **printed page**. And so on.

Picking the right medium first prevents the most common failure mode: **smuggling book-page cues into non-literary content.** Cream paper backgrounds, aged-paper noise textures, serif Latin body, ❖/※/asterism dividers, drop caps, narrow paperback margins — these are *reserved* for literary content. A technical status report on cream paper with a serif Latin body and ❖ ornaments is a category error: it doesn't feel "carefully crafted," it feels confused. A SaaS product launch with editorial paper textures is the same mistake.

Don't blend three styles into mush. Pick one direction and execute with precision:

| Content type | Direction (imitate) | Avoid |
|---|---|---|
| Open-source dev tool README | Documentation site (Stripe / Vercel / Anthropic docs): clean white or true-dark canvas, crisp sans body for Latin, mono for code, structured hierarchy, plenty of negative space | Cream paper, serif Latin body, aged-paper noise, ❖ dividers, drop caps — anything whispering "book" |
| Product or startup launch README | Modern product landing page (Linear / Resend / Plain): bold designed surface, screen-native, often dark or stark-minimal, distinctive display type, asymmetric layouts | Editorial / book-page hybrids, paperback paper textures, "warm paper + dark code" combos, anything that reads as printed |
| Technical project status / audit report | Engineering documentation surface: strong hierarchy, mono numerals, tabular data crisp. A warm off-white background is fine, but the surface stays digital — no paper texture, no aged-paper noise. Latin body should be sans, mono for data. Chinese body can be Noto Serif SC (that's a normal documentation choice for CJK, not a "book" signal on its own) | Aged-paper noise, ❖ ornaments, drop caps, Latin serif body, calligraphic display fonts on every heading — the *accumulation* of these reads as "book", which this isn't |
| **Personal essay / blog post** | **Book-page editorial**: serif body, narrow column (580–680px), generous line-height (~1.9), considered margins, drop caps or ornamental section-opener, classical dividers (❖, ※, asterism, or hairline rules). Should feel like a printed page from a literary magazine or quality paperback — page-like, not webpage-like. A warm cream / rice-paper canvas with subtle radial gradient is often the right move | Webpage chrome, hero CTAs, feature grids, modern landing tropes |
| Recipe / how-to guide | Friendly magazine spread: warm palette, numbered steps as cards, ingredient call-out boxes | Severe / brutalist coldness |
| Course / study notes | Notebook: clear hierarchy, highlight boxes for key concepts, margin annotation feel | — |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YukiOvOb/mdrise](https://github.com/YukiOvOb/mdrise) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
