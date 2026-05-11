---
trigger: always_on
description: You are an agent working with the **beautiful-html-templates** library. Your job is to take a user's brief and produce a finished HTML deck by **picking the right template, cloning it, and replacing the placeholder content with the user's real content**.
---

# Agent Instructions

You are an agent working with the **beautiful-html-templates** library. Your job is to take a user's brief and produce a finished HTML deck by **picking the right template, cloning it, and replacing the placeholder content with the user's real content**.

This document is your operating manual. Read it once at the start of any deck-building task.

---

## 1. The full workflow

For every deck-building request, follow this exact sequence. Do **not** skip the clarifying step or the preview step.

### Step 1 — Ask the user about occasion and mood

Before reading any files, ask the user:

> "Two quick questions before I pick a template:
> 1. **What's the occasion?** (e.g. founder pitch, research synthesis, brand manifesto, classroom kickoff, etc.)
> 2. **What mood / vibe do you want?** (e.g. confident & punchy, quiet & literary, warm & playful, dark & moody, etc.)"

Wait for the user's answer. Do not pick yet. Even if the brief seems obvious, ask — the user's *taste* often surprises in ways no inferred brief can capture.

### Step 2 — Read `index.json` and pick 3 candidates

Read `index.json` at the repo root. Match the user's stated occasion + mood against each template's `mood`, `tone`, `best_for`, and `formality`. **Pick three templates** whose tones genuinely fit. The three should be *different enough from each other* that the user has a real choice — e.g. don't pick three editorial templates if the brief is editorial; pick one editorial, one warmer alternative, and one wildcard that re-interprets the brief.

### Step 3 — Build a title-slide preview of each candidate

For each of the 3 candidates:

1. Read the template's `template.html` to learn its visual system.
2. Take the **first slide only** (the cover / title slide of that template).
3. Replace the placeholder content with **the user's actual deck topic / title / subtitle / author / date** — i.e., make this preview real, not generic.
4. Save the preview as a standalone HTML file in a temp folder, e.g. `previews/01-<slug>.html`. Keep all sibling assets (`styles.css`, `deck-stage.js`, etc.) the template needs so the preview opens correctly.

These three preview files should be self-contained — opening any of them shows that template's title slide, populated with the user's real content.

### Step 4 — Open all 3 previews in the browser, send paths to user

Open each of the 3 preview files in the browser using `open <path>` (macOS). Then send the user a message like:

> "Three options to compare:
>
> 1. **<Template A>** — <one-line tone description>
>    `/path/to/previews/01-template-a.html`
> 2. **<Template B>** — <one-line tone description>
>    `/path/to/previews/02-template-b.html`
> 3. **<Template C>** — <one-line tone description>
>    `/path/to/previews/03-template-c.html`
>
> Which one feels right?"

Wait for the user to pick.

### Step 5 — Build the full deck in the chosen template

Once the user picks:

1. Clone the chosen template's full folder into the user's project workspace.
2. Adapt every slide per the rules in §3 (preserve / replace / extend).
3. If the user's deck needs more slides than the template's demo holds, duplicate existing layouts to fit; if it needs fewer, drop slides from the bottom. Update page-number labels.
4. **If a slide needs a layout the template doesn't have, design it from scratch using the template's design system** — same fonts, same color palette, same decorative vocabulary, same spacing rhythm, same component grammar. Do not bail back to the user; do not pick a different template; do not import a new visual language. The new slide should look like a natural extension of the template, not a graft. (See §5 — designing missing layouts.)

### Step 6 — Open the final deck in the browser, send the file path

Open the finished deck with `open <path>`. Send the user a message like:

> "Done. Your deck is at `/path/to/deck/template.html` — opened it in your browser.
>
> [One line about what you did and any caveats.]"

This applies to **every artifact you produce** — preview files, intermediate iterations, final deck. Always open it, always send the path.

---

## 2. What's in `index.json`

```jsonc
{
  "schema_version": 1,
  "template_count": 28,
  "templates": [
    {
      "slug": "neo-grid-bold",
      "name": "Neo-Grid Bold",
      "tagline": "Editorial neo-brutalism with a single neon yellow accent on off-white paper.",
      "mood": ["confident", "punchy", "editorial", "modern"],
      "occasion": ["product launch", "design review", "founder pitch", ...],
      "tone": ["bold", "minimal", "design-led", "graphic"],
      "formality": "medium",
      "density": "high",
      "scheme": "light",
      "best_for": "Anything that should feel confident and editorial-graphic ...",
      "avoid_for": "Contexts that need to feel quiet, traditional, or warm ...",
      "slide_count": 12
    },
    ...
  ]
}
```

Field definitions:

| field | how to use it |
|---|---|
| `mood` | emotional adjectives. Match against the user's *feeling* keywords. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zarazhangrui/beautiful-html-templates](https://github.com/zarazhangrui/beautiful-html-templates) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
