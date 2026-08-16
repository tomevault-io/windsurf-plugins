---
trigger: always_on
description: Persistent brief for building the **AI Engineering Roadmap** — a public, visual, 15-level learning map that links out to the CampusX platform. Read this fully before writing code, and re-consult it whenever making design or structure decisions.
---

# CLAUDE.md — AI Engineering Roadmap Website

Persistent brief for building the **AI Engineering Roadmap** — a public, visual, 15-level learning map that links out to the CampusX platform. Read this fully before writing code, and re-consult it whenever making design or structure decisions.

---

## 1. What we're building

A standalone marketing-grade website that presents the AI Engineering roadmap as a **visual journey through 15 levels (0–14)**. It is a sibling to the main CampusX site — same brand, calmer/more editorial tone — and its job is to (a) orient a learner on the whole path, (b) let them drill into any level, and (c) route them to the relevant CampusX One courses.

**North stars:** on-brand, visually led (not text-heavy), fast, and obviously sequenced. The *sequence is the product* — the design must make progression legible at a glance.

---

## 2. Content source (do not invent content)

All level content already exists as flat files in `./AI ENGINEER ROADMAP/`. Parse these — never fabricate level content. Filenames are canonical:

```
level0-software-fundamentals.txt      level8-llm-evals.txt
level1-llm-101.txt                    level9-ai-security.txt
level2-llm-orchestration.txt          level10-ai-system-design.txt
level3-prompt-engineering.txt         level11-llmops.txt
level4-embeddings-and-vector-dbs.txt  level12-advanced-ai-systems.txt
level5-rag.txt                        level13-fine-tuning.txt
level6-agents.txt                     level14-projects.txt
level7-context-engineering.txt
```

**Expected content model per level** (parse into this shape; adapt if a file's structure differs, and flag any level that can't be parsed rather than guessing):

- `number` (0–14) and `title`
- `scope` — one-line "what this level is about"
- `outcome` — the "you can…" capability statement
- `modules[]` — each with `name`, `sessionCount`, `objectives[]`
- `stack[]` — tools/tech tags
- `project` — the "ship it" capstone for the level
- `totalSessions` (sum, or stated)

If a level file is prose, extract this structure from it. Keep a single `levels.ts`/`levels.json` as the parsed source of truth so the UI never reads raw `.txt` at runtime.

---

## 3. Tech stack

- **Astro + Tailwind CSS + Lucide icons.** Astro suits a content-driven, mostly-static, visual site and ships almost no JS by default. Tailwind carries utilities; a small design-token layer (below) carries the brand.
- Content collections (or a build-time parse of the `.txt` files) → typed level data.
- Minimal client JS: only for progress tracking, the sticky level navigator, and scroll reveals. No heavy framework runtime.
- If the team prefers hand-authored static HTML (as on Graphy), that's acceptable — but keep the token layer and component conventions identical.

---

## 4. Information architecture

- `/` — **Roadmap overview**: the hero map of all 15 levels, grouped into phases, plus intro, "how to use", and a primary CampusX CTA.
- `/level/[n]` — **Level detail** (deep-linkable), one per level, with prev/next.
- Optional: `/start` (a short "who this is for / how it works" explainer).

**Phase grouping** (use for visual structure and color banding on the map — this is the mental model of the path):

| Phase | Levels | Theme |
|---|---|---|
| Foundations | 0–3 | software, LLM 101, orchestration, prompting |
| Retrieval | 4–5 | embeddings/vector, RAG |
| Agentic | 6–7 | agents, context engineering |
| Evaluation & Trust | 8–9 | evals, security |
| Production | 10–11 | system design, LLMOps |
| Frontier & Proof | 12–14 | multimodal, fine-tuning, projects |

---

## 5. Design system — CampusX theme

Match the CampusX One landing page exactly. It is the canonical style reference. Extract these into `tokens.css` (CSS variables) and mirror in `tailwind.config`.

### Fonts
```html
<link href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;500;600;700&family=DM+Sans:wght@300;400;500;600;700&family=JetBrains+Mono:wght@400;500;600&display=swap" rel="stylesheet">
```
- **Space Grotesk** — display / headings / numbers (700 for headlines, tight tracking).
- **DM Sans** — body copy, buttons.
- **JetBrains Mono** — eyebrows, labels, tags, metadata (uppercase, wide letter-spacing).

### Color tokens
```css
:root{
  /* brand */
  --cx-purple:      #6c5ce7;  /* primary */
  --cx-purple-deep: #4338ca;  /* indigo — gradients, emphasis */
  --cx-purple-hover:#5b4cdb;  /* button hover */
  --cx-violet-700:  #6D28D9;

  /* ink & gray */
  --cx-ink:   #0f0f14;  --cx-ink-2:#18181b;
  --cx-g700:  #3f3f46;  --cx-g600: #52525b;
  --cx-g500:  #71717a;  --cx-g400: #a1a1aa;

  /* surfaces */
  --cx-white: #ffffff;
  --cx-cream: #faf8f3;   /* warm section bg */
  --cx-gray:  #F9FAFB;
  --cx-violet-bg:#FAF9FF; /* light purple tint section */

  /* lines */
  --cx-border:#e7e5e0;  --cx-border-2:#e4e4e7;

  /* success */
  --cx-green:#059669;   --cx-green-bg:#ECFDF5;
}
```

### Pastel pill palette (cyclic — assign per level/module for visual variety)
Each set is `{bg, border, leftAccent}`:
```
violet  #EDE9FE / #DDD6FE / #6D28D9
blue    #DBEAFE / #BFDBFE / #1D4ED8

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [campusx-official/ai-engineering-roadmap-2016](https://github.com/campusx-official/ai-engineering-roadmap-2016) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
