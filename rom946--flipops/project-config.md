---
trigger: always_on
description: These rules are mandatory on every task. No exceptions.
---

# ⚡ COST EFFICIENCY RULES — READ FIRST, ALWAYS FOLLOW

These rules are mandatory on every task. No exceptions.

## @flipops-map anchor rule — apply before editing any file
Check every file you are about to edit for a `@flipops-map` block at L1.

**If the map has line numbers (old format):**
1. Read the existing map block
2. Place inline anchor comments in the code: Python `# ANCHOR:name` / JS/JSX `// ANCHOR:name`
3. Rewrite the map replacing line numbers with `ANCHOR:name` references
4. Remove any OFFSET line — no longer needed
5. Note "Uses named anchors" in the map header
Then proceed with the task.

**If no `@flipops-map` exists (non-test files only):**
1. Complete the task first
2. Add `@flipops-map` with named anchors (no line numbers)
3. Place `ANCHOR:` comments inline in the code

**After completing all changes:**
- Update the `@flipops-map` of every modified file
- Add/update `ANCHOR:` comments as needed
- Never use line numbers in any map

## Before reading any file
1. Check CLAUDE.md first — if the answer is here, do not read any other file
2. Only read files directly named in the task
3. If unsure which files are needed, ask — do not explore speculatively

## File reading rules
- NEVER read node_modules, dist, .vite, __pycache__
- NEVER read lock files (package-lock.json, yarn.lock)
- NEVER read .git folder
- NEVER read image/font/binary files
- NEVER read more than 5 files per task unless explicitly told to
- NEVER re-read a file you already read this session

## Prompt execution rules
- Implement directly — do not plan before implementing unless the task explicitly says "plan first"
- Do not summarize what you are about to do — just do it
- Do not explain what you just did — just confirm done
- Do not ask for confirmation mid-task
- Do not suggest alternatives unless asked
- Do not add features not explicitly requested

## Code editing rules
- Prefer minimal diffs over full file rewrites
- Edit only the specific lines that need changing
- Do not reformat or restyle untouched code
- Do not add comments unless asked
- Do not refactor unless asked
- Do not change variable names unless asked

## Output rules
- After completing a task, output only:
  1. Files modified (list)
  2. What was changed (one line per file)
  3. Any new env variables needed (if any)
- Nothing else — no explanations, no summaries, no suggestions for next steps unless asked

## Session hygiene
- If the conversation exceeds 15 messages, suggest /compact before continuing
- If asked to do more than 3 files in one task, flag it and suggest splitting into subtasks

## Project tracking files

After every batch of completed features, update both files before confirming done:

- **CHANGELOG.md** — append a versioned entry at the top under `[Unreleased]` or a new `[x.y.z]` block
- **STATUS.md** — update "What's working", "In progress", "Known bugs", and "Tech debt" sections to reflect current state

### Changelog entry format
```
## [x.y.z] - YYYY-MM-DD
### Added / Fixed / Changed / Removed
- One line per change, specific (feature name, file, behaviour)
```

### Rules
- Do not write vague descriptions ("improved UI") — name the component and what changed
- Bump patch version (0.x.**z**) for fixes; minor (0.**y**.0) for new features; major (**x**.0.0) for breaking changes
- Do not rewrite existing entries — only append new ones

---

# FlipOps — CLAUDE.md

## 1. Project Overview

FlipOps is a personal AI-powered second-hand market flipping assistant that helps find, analyze, negotiate, and track deals on Wallapop.

Monorepo layout:
```
FlipOps/
├── frontend/    # React + Vite (deployed to GitHub Pages)
├── backend/     # Python Flask (deployed to Render)
├── .github/workflows/deploy-frontend.yml
├── render.yaml
├── CLAUDE.md
└── .gitignore
```

---

## 2. Stack

**Frontend**
- Framework: React 18
- Build tool: Vite 5
- Styling: Tailwind CSS 3 (utility-first; custom component classes in `index.css`)
- Routing: React Router DOM v6 (`BrowserRouter` with `basename="/FlipOps"`)
- i18n: i18next + react-i18next + LanguageDetector (single file: `src/i18n.js`)
- Auth UI: Firebase Auth (Google provider)
- Icons: lucide-react

**Backend**
- Language: Python 3.11+
- Framework: Flask 3 with Flask-CORS
- AI: Anthropic Claude (model: `claude-haiku-4-5-20251001`)
- Scraping: curl_cffi (Chrome impersonation), BeautifulSoup4, ThreadPoolExecutor (8 workers)
- Search: DuckDuckGo HTML queries
- Auth verification: Firebase Admin SDK (ID token)
- Storage: Firebase Firestore
- Encryption: cryptography (Fernet) for API keys at rest

**Database & Auth**
- Firebase Firestore (database)
- Firebase Auth (Google sign-in, ID token verification)

**Deployment**
- Frontend: GitHub Pages (`gh-pages` branch, auto-deployed via GitHub Actions on push to `main`)
- Backend: Render (free tier, `gunicorn app:app`)
- Keep-alive: UptimeRobot pings `/api/health` every 5 min

## Token cost reminders
- `useApi` already handles auth headers — never rewrite auth logic
- Translation is already set up — never add a new i18n library, but always make sure translations are set up correctly on new pages/features you create
- Firestore access pattern is established — follow existing patterns in similar files

---

## 3. Key Files Map

### Frontend (`frontend/src/`)

| Role | Path |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Rom946) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
