---
trigger: always_on
description: Interactive guide for logging engineering work to a personal journal. Use this skill whenever the user wants to record something they built, document a completed project, or reflect on their growth across past work — even when they don't explicitly say "log" or "journal". Trigger phrases include "log what I built today", "record this", "write an entry", "journal this debugging session", "write up the X project", "project snapshot", "project portfolio entry", "growth check", "reflect on the last 
---


# builder-log

A personal engineering journal skill. Helps an engineer record three kinds of things: short entries about individual work moments, comprehensive project snapshots, and zoom-out growth reflections across past records.

All output is plain markdown written into the user's current working directory. No database, no state beyond the files themselves.

## Core principles

These shape every interaction. If a specific step below conflicts with one of these, trust the principle.

**Accept messy input, structure it yourself.** The user should be able to ramble: *"yeah I spent like 3 days debugging a WebSocket memory leak, turned out we weren't closing subscriber refs."* Turn that into Problem / Root Cause / Solution / Lesson. Don't make them learn the template — that's your job.

**One section at a time.** Don't throw the whole template at the user at once. Ask about one thing, get the answer, move on. The reason people abandon journals is the blank page, not the content.

**Skip is fine, faked content is not.** If a section doesn't apply to this entry, mark it skipped and move on. Forcing every field creates hollow journals that don't get reread.

**End with a written file, not a checklist.** Draft the final markdown from the conversation yourself, show it, accept edits, then write. Never leave the user with a template to fill in — that's what failed them before this skill existed.

**Slug-always.** Every entry and growth check must have a clear topic. If the user can't name what it's about, offer to come back later rather than scaffolding an empty file. A vague slug becomes a file nobody rereads.

**Pitch-gate.** Project snapshots require a one-line pitch. If the user can't articulate it after a couple of tries, the project isn't ready to write up yet — offer a stub or park it. Compression exposes understanding; unforced compression exposes its absence.

## Detect the log root

The skill operates on the user's current working directory. Before starting any mode:

1. Check whether CWD contains any of `entries/`, `projects/`, `growth_checks/`.
2. If at least one exists, CWD is the log root. Proceed.
3. If none exist, ask: *"I don't see a builder-log structure here. Create one in this directory?"*
   - On yes: create all three subdirectories and continue.
   - On no: ask where they want to work, `cd` there conceptually (use absolute paths for all subsequent writes).

Never assume a fixed path like `~/builder-log` or any other hardcoded location. The skill must work for anyone who installs it, wherever they run it from.

## Select a mode

Read the user's trigger phrase first. Most map cleanly:

| User says | Mode |
|-----------|------|
| "log what I did", "record this", "journal this", "write an entry" | entry |
| "write up the X project", "project snapshot", "portfolio entry" | project |
| "growth check", "reflect on the last N weeks", "zoom out", "look back at Q1" | growth check |

If the phrasing is ambiguous, ask once: *"Is this a single moment to log, a whole project to write up, or a zoom-out reflection?"* Then commit to the mode and don't second-guess.

---

## Mode: Entry

Purpose: capture a single work moment before it fades — a problem solved, a decision made, something shipped, something learned.

### Step 1 — Slug gate

Ask: *"What's this entry about? One phrase — the topic, not the whole story."*

- If they give a clear topic (e.g., *"WebSocket memory leak"*, *"Postgres migration decision"*): slugify (see [Slug rules](#slug-rules)) and continue.
- If they hesitate or offer vague fluff (*"just stuff I did"*): push back gently. *"Try to name one concrete thing — a problem, a decision, a moment you'd want to remember. If nothing sticks out, this might not be ready to write yet."*
- If they still can't name it: offer to skip, and suggest coming back when something concrete surfaces. Don't scaffold empty entries.

### Step 2 — Filename and collision check

Compute `entries/YYYY-MM-DD-<slug>.md` using today's date.

If a file with that exact name already exists, ask whether this is a continuation of that entry or a separate one. On separate, append `-part-2` or a short differentiator.

### Step 3 — Triage

Ask: *"Quick note or longer reflection?"*

- **Quick**: walk only the sections that clearly matter (usually What I Built, or Problems Solved + Learned). Two or three questions, a couple of minutes total.
- **Longer**: walk the full set of template sections, skipping gracefully where nothing applies.

### Step 4 — Conversational walkthrough

Read `assets/entry_template.md` to get the current section names and structure. Then ask one section at a time, conversationally. Suggested prompts:

- *What did you build or ship?*
- *Any calls you had to make — between approaches, libraries, designs?*

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [C0ldSmi1e/builder-log-skill](https://github.com/C0ldSmi1e/builder-log-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
