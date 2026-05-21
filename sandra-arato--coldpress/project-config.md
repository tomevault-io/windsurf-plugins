---
trigger: always_on
description: Operating instructions for producing whimsical, retro-European interactive children's travel scrapbooks/activity journals. One brief in → one finished, print-ready scrapbook out.
---

# P8 — Children's Travel Scrapbook Factory

Operating instructions for producing whimsical, retro-European interactive children's travel scrapbooks/activity journals. One brief in → one finished, print-ready scrapbook out.

This file is **runtime spec**, not status. PARA status, Hemingway bridge, and dated notes live in sibling files.

---

## Status

- **Active book:** `books/hungary-2026/`
- **Deadline:** 2026-06-01 (print lead time before Jun 6 departure)
- **Pipeline stage:** ✅ Spec locked → ✅ Research locked → ✅ Editorial plan locked → ✅ Copy locked → ✅ Layouts locked → ✅ Illustrations locked → **→ Affinity assembly (next)**
- **Recraft credits:** ~2,480 / 8,000 used

## Hemingway Bridge

2026-05-07 [21:00] · Thursday — huge production day. All content tracks locked for `hungary-2026`. Spec, research, red-flag decisions, visual decisions, editorial plan (38pp: 4/15/8/8/3 split), copy (7 files, Hungarian throughout), layout specs (all sections + cover + sticker sheets), and 37 illustration assets are done. Recraft drama: trained custom style (`c9b1c5bb-…`, V3) underperforms V4 base for most assets — locked feedback memory. V3 kept only for 6 strong Budapest landmarks. 2,480/8,000 Recraft credits used. · Next: Canva Producer sub-agent — assemble book from locked assets. Start with Intro section (pp.1–4) to validate the full-bleed-to-Canva-frame workflow before full run. Canva MCP tools must be working: verify `mcp__canva__*` connection before firing the sub-agent. Deadline Jun 1 is 25 days out — Canva assembly is the only remaining track. · Blockers: none structural. If Canva MCP is unavailable, escalate immediately — this is a hard dependency for the final production phase.

2026-05-08 [late] · Switched from Canva to Affinity for the assembly track. Built affinity-design skill at .claude/commands/affinity-design/. Producer agent renamed scrapbook-canva-producer → scrapbook-affinity-producer; designer's Canva MCP tools swapped for Affinity. Next: dry-run the new Producer on the Intro section (pp.1–4) before full book.

---

## 1. Mission

Take a creative brief for a children's travel scrapbook and produce a print-ready interactive activity journal through a coordinated multi-agent pipeline. The system must generalise across briefs — no brief-specific details (ages, languages, locations, palettes) are hardcoded into agents or instructions.

The aesthetic baseline is editorial children's publishing: flat geometric vector illustration, retro travel-poster influence, calm structured layouts, light cultural depth. **Avoid:** watercolor, kawaii, clipart, AI-looking imagery, dense workbook formatting.

---

## 2. The Scrapbook Spec

Every book is described by a structured spec. The main agent fills this in from the brief + clarifying questions before any sub-agent fires.

| Field | Description |
|---|---|
| `book_slug` | Short kebab-case ID, e.g. `hungary-2026` |
| `audience` | List of children: age, reading/writing level, languages spoken, interaction modes (sticker, circle, draw, write, glue) |
| `language` | Book content language(s); monolingual or bilingual layout rules |
| `locations` | Ordered list of regions/cities/topics; one becomes the primary section |
| `page_count` | Target total (e.g. 50–60) and rough split per section |
| `format` | Page size, orientation, binding, finish, print target (home/professional) |
| `style` | Illustration style, palette directions per section, typography stance, density, references to emulate, references to avoid |
| `activities` | Recurring activity systems in scope (observation, ratings, guided drawing, glue-in, reflection, scavenger hunts, conversation prompts, map tracking) |
| `cultural_depth` | Light-touch / moderate / heavy. Default: light-touch and integrated |
| `production` | Print constraints: ink-consciousness, marker/pencil friendliness, background fill rules |
| `extras` | Sticker sheets, fold-outs, pockets, cover, colophon |
| `deadline` | Target use date and lead time for printing |

The spec lives at `{book_slug}/spec.md`. Lock it before production starts.

---

## 3. Pipeline

Linear with explicit human checkpoints. Do not skip a checkpoint to save time.

```
Brief intake
  ↓
Clarifying questions → Spec draft → [HUMAN: spec lock]
  ↓
Research (Researcher + Visual Scout in parallel) → [HUMAN: research review]
  ↓
Editorial plan: page-by-page outline (Editor) → [HUMAN: plan lock]
  ↓
Copy drafts (Copywriter) ⊕ Illustration prompts + generation (Illustrator) ⊕ Layout specs (Designer)
  ↓
Editor pass → revisions → [HUMAN: content lock]
  ↓
Affinity assembly (Affinity Producer)
  ↓
Final vision pass (Editor + Visual Scout) → [HUMAN: ship/no-ship]
  ↓
Export → handoff
```

Run the three middle tracks (copy, illustration, layout) in parallel where possible — they coordinate through the editorial plan, not through each other.

---

## 4. Brief Intake & Clarifying Questions

When a brief arrives:

1. Read it fully. Read any attached imagery using vision (the Read tool on image paths).
2. Draft the spec from the brief. For each field, mark confidence:
   - **≥ 0.85** — fill it in, flag with `⚠️ confirm`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sandra-arato/coldpress](https://github.com/sandra-arato/coldpress) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
