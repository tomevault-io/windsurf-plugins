---
trigger: always_on
description: A personal 2026 learning challenge: one new AI topic learned and one new piece of software built
---

# CLAUDE.md

## Project Overview

A personal 2026 learning challenge: one new AI topic learned and one new piece of software built
every week. Weeks are numbered 1–52, each starting on a Monday.

- `wiki/` — LLM-maintained knowledge base tracking all learning and builds
- `code/` — all builds, organized by domain (data-structures/, gen-ai/, databases/, etc.)

The human's job: source material, direction, questions.
The LLM's job: all bookkeeping — filing, cross-referencing, maintaining consistency.

---

## Human-Driven Workflows

These are the tasks the human drives. Use natural language — the phrases below are examples,
not rigid commands.

### Start a week's AI learning
Triggers: "start week N AI learning" / "new AI learning for week N" / "set up week N learning" / "start a new AI learning" / "new AI learning"

If a specific week number N is given, use it directly.

If no week number is given, determine the target week from repo state first:
1. Run the Current week status workflow to identify the in-progress week (if any) and the last completed week
2. If a week is **in progress** (scratch files exist beyond the last ingested week): do not assume — ask the human "You have Week N in progress — should I add this AI learning to Week N, or start it as part of a new week?"
3. If no week is in progress: target the next week after the last completed one

Then:
1. Look up the nominal start date for the target week from the Week Reference table below
2. Create `wiki/scratch/week-NN-yyyy-mm-dd-ai-learning.md` with this header:
   ```
   # Week NN — AI Learning
   Topic:
   Started: <today's actual date>
   Week date: <nominal Monday date>

   ---

   ```
3. Tell the human the exact file path so they can open it and start writing freely

### Start a week's build
Triggers: "start week N build" / "new build for week N" / "set up week N build" / "start a new build" / "new build"

If a specific week number N is given, use it directly.

If no week number is given, determine the target week from repo state first:
1. Run the Current week status workflow to identify the in-progress week (if any) and the last completed week
2. If a week is **in progress** (scratch files exist beyond the last ingested week): do not assume — ask the human "You have Week N in progress — should I add this build to Week N, or start it as part of a new week?"
3. If no week is in progress: target the next week after the last completed one

Then:
1. Look up the nominal start date for the target week from the Week Reference table below
2. Create `wiki/scratch/week-NN-yyyy-mm-dd-build.md` with this header:
   ```
   # Week NN — Build
   What I'm building:
   Started: <today's actual date>
   Week date: <nominal Monday date>

   ---

   ```
3. Tell the human the exact file path so they can open it and start writing freely

### Start both for a week
Triggers: "start week N" / "set up week N" / "begin week N" / "start a new week" / "new week"

If a specific week number N is given, use it directly. If no week number is given, apply the same repo-state logic as above (check in-progress, confirm with human if ambiguous, otherwise use next week after last completed).

Run both the AI learning and build workflows above in sequence. Create both scratch files and confirm both paths.

### Current week status
Triggers: "what week am I on" / "which week am I on" / "what week is this" / "where am I in the project"

Do NOT answer with the current calendar week number. Instead, determine the project week from the repo state:

1. Check `wiki/weeks/` for the highest-numbered ingested week page — that week is **complete**
2. Check `wiki/scratch/` for any week files beyond the last ingested week — that week is **in progress**
3. Report accordingly:
   - If a scratch file exists beyond the last ingested week: "You're currently on Week N (in progress)"
   - If no scratch file exists beyond the last ingested week: "Week N is complete — Week N+1 is next"
   - If no weeks/ pages exist yet: "No weeks completed yet — Week 1 is next"

### Add to backlog
Triggers: "add to backlog: [idea]" / "backlog this: [idea]" / "I want to learn X" / "add build idea: [idea]"

1. Determine which section the idea belongs to (AI Learning Ideas or Build Ideas) from context
   — if unclear, ask
2. Append it to the appropriate section in `wiki/backlog.md`
3. Confirm what was added and to which section

---

## Wiki Operations

### Ingest
Triggers: "ingest week N" / "ingest weeks N and M" (for catch-up)

1. Look up the nominal start date for week N from the Week Reference table below
2. Read `wiki/scratch/week-NN-yyyy-mm-dd-ai-learning.md` and `wiki/scratch/week-NN-yyyy-mm-dd-build.md`
3. Create `wiki/weeks/week-NN-yyyy-mm-dd.md` — overview page with frontmatter, linking to tools/ and builds/ pages
4. Create or update the appropriate page in `wiki/tools/` for the AI topic covered
5. Create a new page in `wiki/builds/` for what was built — named descriptively and distinctly
6. Create or update any `wiki/concepts/` pages for cross-cutting ideas that emerge
7. Remove from `wiki/backlog.md` any items matching what was just learned or built
8. Update `wiki/index.md` — add new pages, update summaries of modified pages

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RGirish) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
