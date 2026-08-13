---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repo structure

This is a monorepo of unrelated small projects, not a single application:

- `data-hub/` — React + Vite dashboard ("DATA POPCORN"). Single-page app (`src/App.jsx`) that renders a sidebar of data categories (external systems like Haerapy/Jeju Golf/restaurants, plus a personal hub: blood donation, diet, OOTD, environment, health, finance) and a generic table view driven off `src/sampleData.js`. All data is currently hardcoded sample data — there is no backend wired up; adding a new category means adding a key to `sampleData.js` and an entry to the `categories`/`personalData` arrays in `App.jsx`.
- `passid/` — React + Vite + Tailwind v4 landing page + editor for an AI ID-photo product ("PassID"). `src/App.jsx` is the marketing page; clicking "Start for Free" swaps to `src/components/Editor.jsx`, which uploads a photo and runs client-side background removal via `@imgly/background-removal` (WASM/GPU, runs entirely in-browser — no server upload). AI generation prompts (Korean-language, for external tools like Gemini) live in `src/constants/prompts.js` and are treated as a distinct, curated asset — edit the prompt text carefully, it's tuned for identity-preserving ID photo output.
- `neis-meal-api/` — NEIS Open API (Korean school meal/school info) scripts, decoupled from the three apps above:
  - `fetch_schools.py` — samples schools per region from the `schoolInfo` NEIS endpoint into `schools.json` (relative to this folder).
  - `fetch_meals_paginated.sh` — bash/curl pagination loop against the `mealServiceDietInfo` endpoint into `meals_all_pages.json` (relative to this folder).
  - `meal_pagination_workflow.json` / `school_data_workflow.json` — n8n workflow exports mirroring the same fetch logic as n8n nodes (manualTrigger → set/code → httpRequest → splitInBatches/code). Treat these as the canonical workflow definitions if editing the n8n automations; the shell/Python scripts are standalone equivalents, not generated from them.
  - `AGENT.md` / `MEAL_API_GUIDE.md` — reference docs for the NEIS meal API (params, response shape, allergy codes). Consult these before changing any meal-fetching code instead of guessing param names.
- `public-dataset/` — practice-data folders for AI tool education (Claude Code, Codex, etc.), grouped under one directory: `pdf/`, `csv/`, `excel/`, `ppt/`, `image/`, `movie/`, `json/`, `audio/`, `txt/`, `zip/`, `sqlite/`, `html/`. Not wired into any app. Each subfolder has generated sample files plus a `README.md` describing what it's for and example exercises; `movie/` holds a link list (`sample_video_links.md`) to public sample mp4s instead of large video files. `ppt/` exists for one exercise: **design reproduction** (hand Claude an existing deck, get a new one in the same design), not summarizing. It holds a deliberate pair — `ictk_deck_template.pptx` (PptxGenJS-generated in-house template, 15 slides, 10 body layouts, coordinates on an exact 1920×1080px grid at px×0.75=pt, so its rules are easy to reverse-engineer) and `creative_brief_deck.pptx` (a downloaded Canva free template, hand-built, photo-driven). Work the generated one first. Each has a measured `<deck>.DESIGN.md`; `DESIGN-METHOD.md` holds the reusable procedure/script/prompt that produced both, so if a deck is swapped, re-run that script rather than hand-editing numbers. The flagship exercise pairs `business-docs/references/지난제안서.md` (content) with the ICTK template (design) — the README carries the section→layout mapping, prompt, and pass criteria, and `example_proposal_deck.js`/`.pptx` is the worked answer (11 slides from 9 sections, kickers written rather than copied, tables via `addTable()` so they stay editable, common chrome factored into one `chrome()` helper). Regenerate the pptx by running the script rather than editing the binary. Both decks are practice-only: the Canva license restricts redistributing the template file itself, and the ICTK deck still carries that client's branding (its placeholders hold no real IR data — an earlier `ICTK_IR_Book_2025.pptx` was rejected for this folder because slide 2 was an investor disclaimer forbidding redistribution). Re-encode embedded images (1920px / q80) before committing any further deck: the Canva raw download was 39MB and git history is permanent. One deliberate exception to the by-file-type grouping: `business-docs/` is a **lecture practice set** that mirrors the student's working-folder layout (`references/`, `drafts/`, `data/`) instead of splitting by type — the B2B lecture prompts point at paths like `@references/지난제안서.md`, so the folder structure itself is the payload. Keep those files where they are and keep their section headings explicit (the structure-extraction exercises depend on them); everything in there is mock data (fictional company 주식회사 세이프칩, fictional people/customers/amounts/dates).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [team-datapopcorn/datapopcorn-datahub](https://github.com/team-datapopcorn/datapopcorn-datahub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
