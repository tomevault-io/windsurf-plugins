---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

Magic Slide is a Codex skill that generates polished, self-contained HTML presentations with Magic Move transitions. The skill uses a structured workflow with user checkpoints at search and outline stages.

## Core Architecture

### Generation Model

**Multi-phase workflow with user confirmation:**
Before step 1, create and maintain a visible TODO/plan for every
`$magic-slide` invocation; the detailed rule lives in `SKILL.md`.
1. Ask user about topic, aesthetic style, language, images
2. Ask if user wants web search (optional). If yes, run `scripts/websearch.py`
   first; agent/default web search is only a fallback after the script path
   fails or returns no usable results.
3. Generate outline with a Magic Move spine and get user confirmation (REQUIRED)
4. Write Brief Lite, including Magic Move content-relay motion grammar, in the conversation before coding
5. Read reference files, make a compact internal style/layout plan with primary/supporting Magic Move continuity, and generate all modular HTML sources directly
6. Merge slides into single HTML
7. Inject FLIP engine and runtime
8. Launch the Magic Slide preview server, capture one Playwright QA overview
   longshot first for newly generated decks, fix obvious rendered issues
   visible in the overview, then stop for the mandatory user `Revise slide`
   marking pass before final repairs and delivery. Do not run single-slide
   screenshot repair before that pause. When resuming from saved
   `visual-issues.json` notes, read JSON and source files first, repair marked
   slides, then mark those JSON records fixed and awaiting user confirmation.
   Do not run screenshot verification after repairing saved JSON notes unless
   the user explicitly asks.

**Why this works:** User controls information gathering and reviews structure before generation. Brief Lite gives the deck an art direction without returning to the old long prototype loop. Read design guidelines once, generate all slides in main thread. Fast and simple with clear checkpoints.

### FLIP Animation Engine

Elements with matching `data-magic-id` animate smoothly between slides using the FLIP (First, Last, Invert, Play) technique. The engine is auto-injected by `inject-runtime.py`.

**Critical requirements:**
- Text elements with magic-id MUST have `display:inline-block` style
- Only assign magic-id where visible text is IDENTICAL on both slides
- No decorative elements should have magic-id

## Build Pipeline

### Core Scripts

All scripts are in `scripts/` directory:

**Generation & Assembly:**
- `merge-slides.py` — Combines modular sources into `[topic]/index.html`
- `inject-runtime.py` — Adds FLIP engine, navigation, progress bar, stagger animations
- `generate-image.py` — AI image generation via PipeLLM API
- `websearch.py` — Web search via PipeLLM WebSearch API

**Preview:**
- `serve.py` — Single-service preview server with deck-scoped routes

**Utilities:**
- `extract-slides.py` — Decomposes merged HTML back to modular sources
- `mark-qa-repaired.py` — Marks saved QA revision notes fixed and awaiting
  user confirmation without running screenshot verification

### Build Flow

```
1. Ask about web search (AskUserQuestion)
2. If user chooses yes, use scripts/websearch.py (PipeLLM WebSearch API) before
   any agent/default web search fallback
3. Generate sources/outline.md with Magic Move spine and confirm with user (AskUserQuestion - REQUIRED)
4. Write Brief Lite with Magic Move content-relay motion grammar in the conversation
5. Read reference files (generation-guide.md, layout-guide.md, html-contract.md, design-system.md, flip-engine.md; images.md when needed)
6. Read sources/outline.md
7. Generate a compact internal style/layout plan with primary/supporting Magic Move continuity, then create sources/ files directly (style.css, slide-01.html, slide-02.html, ...)
8. merge-slides.py combines them into index.html
9. inject-runtime.py adds FLIP + navigation to index.html
10. serve.py launches preview and remains running for QA/editing
11. For newly generated decks, open `?ms_qa=overview&ms_qa_capture=1`, capture
    one Playwright full-page scrolling QA overview longshot, fix obvious
    rendered issues visible in the overview, then stop for the mandatory user
    `Revise slide` marking pass. Do not run single-slide screenshot repair
    before that pause.
```

### Running Scripts

```bash
# Merge slides
python3 scripts/merge-slides.py [topic]/sources/ --lang [language]

# Inject runtime
python3 scripts/inject-runtime.py [topic]/index.html --lang [language]

# Preview (MUST use this, not python3 -m http.server or file://)
python3 scripts/serve.py [topic]/index.html
```

Do not finish a deck-generation or deck-update task until the preview server is
running and the user has the displayed URL. The in-browser editor depends on
`scripts/serve.py`; opening the HTML file directly disables server-backed Save,
image replacement, and close/shutdown behavior.

For newly generated decks, delivery has an extra required pause: after the
agent captures one Playwright QA Overview longshot and fixes the most obvious

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daniel-style/magic-slide](https://github.com/daniel-style/magic-slide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
