---
trigger: always_on
description: **Shlav A Mega** is a Progressive Web App (PWA) for Israeli geriatrics board exam preparation (שלב א גריאטריה, P005-2026). It is a single-file, no-build-step application deployed via GitHub Pages.
---

# CLAUDE.md — Shlav A Mega: Israeli Geriatrics Board Exam App

## Project Overview

**Shlav A Mega** is a Progressive Web App (PWA) for Israeli geriatrics board exam preparation (שלב א גריאטריה, P005-2026). It is a single-file, no-build-step application deployed via GitHub Pages.

- **Live URL**: https://eiasash.github.io/Geriatrics/
- **Main file**: `shlav-a-mega.html` (~236 KB, ~3,800 lines, self-contained HTML/CSS/JS)
- **App version**: v9.10
- **Data**: JSON files in `data/` directory, loaded lazily at runtime
- **Deployment**: Push to `main` → GitHub Actions validates → GitHub Pages live in ~60s

---

## Architecture

### Single-File PWA

All application logic lives in `shlav-a-mega.html` — no bundler, no framework, no build step. The file contains:
- All CSS (1,000+ lines, responsive, RTL-aware, dark/light/study modes)
- All JavaScript (ES6+, vanilla)
- HTML structure

Data is loaded at runtime from `data/*.json` files. The service worker (`sw.js`) caches all assets for offline use.

### Storage Layers

| Layer | Keys / Table | Purpose |
|-------|-------------|---------|
| `localStorage` | `samega`, `samega_ex`, `samega_apikey` | User preferences, exam state, API key |
| `IndexedDB` | (internal) | Study progress, spaced repetition state |
| Supabase PostgreSQL | `progress_state` (RLS) | Optional cloud sync across devices |

**Important**: localStorage keys `samega`, `samega_ex`, `samega_apikey` must not be renamed — they are stored in users' browsers.

---

## File Map

```
/
├── shlav-a-mega.html        # Main app (THE file — all HTML/CSS/JS, v9.10)
├── index.html               # GitHub Pages redirect → shlav-a-mega.html
├── sw.js                    # Service worker (offline caching + background sync)
├── manifest.json            # PWA manifest
│
├── data/                    # Lazy-loaded JSON data — single source of truth
│   ├── questions.json       # 1,419 MCQs (primary runtime source)
│   ├── notes.json           # 40 study topic notes
│   ├── drugs.json           # 53 Beers/ACB drugs database
│   ├── flashcards.json      # 159 high-yield flashcards
│   ├── osce.json            # OSCE station scenarios
│   ├── tabs.json            # Tab definitions for app navigation
│   └── topics.json          # 40 topic keyword mappings for auto-tagging
│
├── explanations_cache.json  # Pre-generated AI explanations (2.3 MB)
├── hazzard_chapters.json    # Hazzard's 8e textbook content (structured JSON)
│
├── questions/               # Question images for exams with figures
│   ├── image_map.json       # Maps question IDs to image files
│   └── images/              # PNG images referenced by exam questions
│
├── scripts/
│   ├── generate_explanations.cjs   # Bulk explanation generator (Claude API)
│   └── parse_2025_exam.cjs         # PDF → JSON question parser
│
├── skill/
│   ├── SKILL.md             # Geriatrics knowledge skill package for Claude Projects
│   └── references/
│       ├── exam-patterns.md # Repeating question stems and frequencies
│       └── legal-ethics.md  # Israeli law summaries
│
├── laws/                    # Israeli legal/regulatory documents
│   ├── P005-2026-syllabus.pdf
│   ├── dying_patient_law.html
│   ├── driving_report_form.docx
│   └── ...                  # MOH/MoJ PDFs and legal references
│
├── .claude/
│   ├── launch.json          # Dev server: python -m http.server 3737
│   ├── agents/              # Agent workflow prompts (note-updater, question-explainer)
│   ├── commands/            # Slash command definitions (see Skills section)
│   └── skills/              # Skill files (shlav-a-mega.md, supabase)
│
├── .github/
│   └── workflows/ci.yml     # Validation CI — JSON schema, duplicates, version sync, etc.
│
├── tests/
│   ├── dataIntegrity.test.js  # 30 tests: question schema, duplicates, topic coverage
│   └── appIntegrity.test.js   # 11 tests: HTML structure, SW sync, security
│
├── supabase-setup.sql        # Supabase RLS schema
├── .mcp.json                 # MCP server config (Supabase)
│
├── harrison/                 # Harrison's 22e chapter PDFs (~48 chapters)
├── hazzard_marked/           # Hazzard's 8e annotated/marked chapter PDFs
├── article_*.pdf             # 6 mandatory clinical reference articles
└── hazzard_part*.pdf         # Hazzard's Geriatric Medicine 8e (original PDFs)
```

### Data Architecture (v9.10)

All runtime data lives in `data/`. The app and service worker load exclusively from `data/*.json`. Build scripts (`scripts/`) also read/write `data/questions.json` directly. There are no root-level JSON duplicates — `data/` is the single source of truth.

---

## Data Schemas

### questions.json
```json
{
  "q": "Question text (Hebrew or English)",
  "o": ["Option A", "Option B", "Option C", "Option D"],
  "c": 0,       // correct answer index (0–3, integer)
  "t": "2022",  // exam year string
  "ti": 18,     // topic index (0–39, see TOPICS below)
  "e": "..."    // optional pre-generated AI explanation
}
```

### notes.json
```json
{
  "id": 0,
  "topic": "Biology of Aging",
  "ch": "Hazzard's Ch 3 (Biology of Aging)",  // MUST cite Hazzard's 8e or Harrison's 22e chapter — NO GRS

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Eiasash) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
