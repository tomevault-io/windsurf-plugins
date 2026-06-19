---
trigger: always_on
description: AI-powered job search and application pipeline. Scans portals (LinkedIn, Indeed, Glassdoor, Greenhouse, Ashby, Lever, Workable, Welcome to the Jungle, Handshake, Reed UK), scores postings against the candidate's profile with A–G blocks, drafts tailored CVs and cover letters, runs interview prep with STAR+R stories, and logs everything to a Notion Applications tracker (auto-created on first run) plus a Notion dashboard view. The agent NEVER auto-submits — it stops before Submit/Send/Apply and wai
---


# Career-Ops — AI Job Search Pipeline

## What this skill does

```
scan portals → score postings (A–G) → draft tailored CV + cover letter → human approves → submit (you click) → Notion tracker logs everything
```

The agent **never auto-submits**. It fills forms, drafts answers, generates PDFs, prepares cover letters — but always stops before Submit / Send / Apply and waits for the user to confirm.

## When invoked

Default response to:

- The user types `/career-ops` (with or without arguments).
- The user pastes a job URL or JD into the chat.
- The user asks to "scan jobs", "find roles", "tailor my CV", "draft a cover letter", "evaluate this job", "prep for an interview at X", "show my pipeline", "log this application".

---

## Where the skill lives vs. where user data lives

The skill is **self-contained**. After install (`git clone https://github.com/marz1307/career-ops ~/.claude/skills/career-ops && cd ~/.claude/skills/career-ops && npm install`), this layout is on disk:

```
~/.claude/skills/career-ops/          ← ENGINE_DIR (this skill folder)
├── SKILL.md                          ← you are reading this
├── modes/*.md                        ← mode definitions
├── templates/                        ← portal config, CV template, states
├── config/profile.example.yml
├── *.mjs                             ← scan.mjs, generate-pdf.mjs, …
├── package.json                      ← node deps (playwright, js-yaml, dotenv)
└── node_modules/                     ← created by `npm install`
```

ENGINE_DIR is **read-only from the user's perspective**. Updates land here; nothing user-specific is written here.

The **WORKSPACE** is the directory the user runs `/career-ops` from. That's where personal files go:

```
<wherever the user cd'd to>          ← WORKSPACE
├── cv.md
├── article-digest.md
├── config/profile.yml
├── modes/_profile.md
├── portals.yml
├── .env                              ← BRIGHTDATA_API_KEY, NOTION_TOKEN
├── data/                             ← applications.md, pipeline.md
├── reports/
├── output/                           ← generated PDFs
└── interview-prep/
```

**Convention used throughout this skill:**
- "Read `modes/X.md`" → read from **ENGINE_DIR/modes/X.md**.
- "Read / write `cv.md` / `config/profile.yml` / `portals.yml` / `data/*` / etc." → operate on the **WORKSPACE** copy.
- Bash invocations like `node scan.mjs` → `node $ENGINE_DIR/scan.mjs` with cwd = WORKSPACE so the script reads the user's portals.yml and writes to data/.

To get ENGINE_DIR, the agent uses the absolute path of this SKILL.md (Read tool resolves it). On most systems it's `${HOME}/.claude/skills/career-ops` (POSIX) or `${USERPROFILE}\.claude\skills\career-ops` (Windows).

---

## Step 1 — Onboarding check

The WORKSPACE = the user's current working directory.

Silently check whether the user-layer files exist in the WORKSPACE:

| Required file / state | If missing |
|---|---|
| `cv.md` | Enter onboarding (Step 2). |
| `config/profile.yml` | Enter onboarding (Step 2). |
| `modes/_profile.md` | Copy `$ENGINE_DIR/modes/_profile.template.md` → `WORKSPACE/modes/_profile.md` silently. |
| `portals.yml` | Enter onboarding (Step 2). |
| `.env` with `NOTION_TOKEN` | Enter onboarding (Step 2). |
| `config/profile.yml → notion.applications_db_id` | Enter onboarding (Step 2). |

If ALL exist, jump to Step 6 (route the user's request).

Before starting onboarding, confirm the workspace location:

> "I'll set up your career-ops workspace in `${cwd}`. That's where your CV, profile, tracker, and generated PDFs will live. Reply 'yes' to use this folder, or give me a different absolute path."

If the user gives a different path: `mkdir -p <path>`, then use it as WORKSPACE for the rest of the session.

## Step 2 — Onboarding: collect inputs

Use **AskUserQuestion** to collect the onboarding inputs. The tool caps at 4 questions per call, so split into two consecutive batches:

> "Welcome to career-ops. To set up your personal pipeline I need a few things. Nine questions split across three batches (4 + 4 + 1)."

**Batch 1 (questions 1–4):**

1. **CV** — `header: "Your CV"`. Multi-select: false. Options:
   - "Paste my CV text in the next message"
   - "Give me a file path (PDF / DOCX / MD) on this machine"
   - "Use my LinkedIn profile only"
   - "Draft from scratch — I'll answer questions"

2. **LinkedIn** — `header: "LinkedIn"`. Multi-select: false. Options:
   - "I'll paste my LinkedIn URL in the next message"
   - "I don't have a LinkedIn / skip"

3. **Portfolio** — `header: "Portfolio"`. Multi-select: false. Options:
   - "I'll paste my portfolio / personal site URL in the next message"
   - "I'll paste my GitHub URL in the next message"
   - "Skip — no portfolio"

4. **Target markets** — `header: "Markets"`. Multi-select: **true**. Options:
   - "United Kingdom"
   - "European Union (broad)"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marz1307/career-ops](https://github.com/marz1307/career-ops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
