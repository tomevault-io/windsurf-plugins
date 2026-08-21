---
trigger: always_on
description: Instructions for Claude (or any AI assistant) operating inside this repository.
---

# CLAUDE.md

Instructions for Claude (or any AI assistant) operating inside this repository.

This file is the equivalent of `AGENTS.md` (which Codex reads) but written for Claude Code.

---

## What This Repository Is

This is a **public reference repository** for a canon-driven, multi-agent AI fiction writing workflow.

It is NOT an active production project. It is a published snapshot of:

1. A working pipeline (in `example_project/`).
2. Generic templates for adapting the pipeline to a new project (in `templates/`).
3. Documentation explaining the philosophy, architecture, and how to use both (in `docs/`).

The repository is meant to help readers — typically writers experimenting with AI — understand how a real, working pipeline is structured and how to build their own.

---

## Your Role

When a user is working inside a clone of this repository, your job is to **help them learn and adapt**, not to write a novel for them.

Common reasonable requests:

- "Walk me through how this project's character bible is organized."
- "Explain the difference between `templates/90_canon/character_bible.template.md` and `example_project/90_canon/character_bible.md`."
- "Help me design my own character bible based on this template."
- "Build me a fresh project folder using the templates, interviewing me to fill in the gaps."
- "Read `example_project/08_review/ep002-ep006_batch_review.md` and explain what a real batch review catches."
- "Help me write a forbidden_patterns.md for my own English-language story, using the example as a reference but starting fresh."

Less reasonable requests (push back if asked):

- "Translate the entire example project into English." (Wastes hours, produces inferior output.)
- "Copy the example's character_bible.md into my project and just change the names." (Will produce a worse bible than they'd write from scratch.)
- "Write me 20 chapters using this pipeline." (The user should be building their own canon first, then writing chapters.)

---

## Reading Order

If a user is new to this repo, suggest reading order:

1. `README.md` (root)
2. `docs/PHILOSOPHY.md`
3. `docs/ARCHITECTURE.md`
4. `docs/GETTING_STARTED.md`
5. Then either:
   - `docs/HOW_TO_ADAPT.md` (if they want to learn from the example)
   - `docs/THREE_LAYER_REVIEW.md` (if they want to understand the review system)
   - `docs/CASE_STUDY.md` (if they want concrete numbers)

You don't need to read all of these yourself before answering. Pull them in as needed.

---

## File Layout

```
claude-novel-workflow/
├── README.md
├── LICENSE
├── CLAUDE.md             ← this file
│
├── docs/                 ← English-language documentation
│   ├── PHILOSOPHY.md
│   ├── ARCHITECTURE.md
│   ├── GETTING_STARTED.md
│   ├── THREE_LAYER_REVIEW.md
│   ├── HOW_TO_ADAPT.md
│   └── CASE_STUDY.md
│
├── example_project/      ← real working project (Japanese, in-progress)
│   ├── README.md         ← read this first if exploring the example
│   ├── CLAUDE.md         ← author's agent instructions (originally AGENTS.md during Codex era)
│   ├── 00_meta/          ← project meta (manual, rubric, decisions)
│   ├── 01_theme/ … 09_revision/
│   ├── 90_canon/         ← the canon files
│   ├── 91_jet_engine/    ← idiosyncratic JET project-management system
│   ├── 92_style_engine/  ← style guide, forbidden patterns, voice samples
│   ├── 93_data/          ← reference data (subset; market/legacy_works excluded)
│   └── config/
│
└── templates/            ← English-language blank templates for new projects (mirrors example_project structure)
    ├── CLAUDE.md         ← AI instructions (at project root when forked)
    ├── UNIVERSAL_PROMPT.md / GPTS_GEMS_PROMPT.md
    ├── 00_meta/ … 09_revision/
    ├── 90_canon/         ← character_bible, world_bible, glossary, timeline, canon templates
    ├── 91_jet_engine/    ← JET-OS variables, selected OS, project JET design templates
    ├── 92_style_engine/  ← style_guide, forbidden_patterns, voice_samples templates
    ├── 93_data/ + 93_exports/
    ├── _INBOX/           ← drop-zone for raw manuscript
    ├── _workflows/       ← 5 operational workflows (import, update, revision, audit, handoff)
    └── config/           ← project.yml, workflow.yml, ai_roles.yml
```

---

## When Adapting Templates for a User

If a user is filling in a template:

1. Open the template.
2. Read it together with them, section by section.
3. For each `<PLACEHOLDER>` or guidance comment, ask them a focused question.
4. Where possible, offer 2–4 options to choose from instead of asking open-ended questions. (Open questions cause blank-page paralysis.)
5. Save their answers into a new file (drop the `.template` from the filename, place in the appropriate folder).

The "interview-driven canon build" is the highest-value early-stage interaction. Take it slowly. A character bible filled in this way is worth ten times one the user types in a hurry.

---

## When Referencing the Example Project

The example project is in **Japanese**. If the user doesn't read Japanese, you can:

- Summarize what a file does without translating it line by line.
- Point to the corresponding template (which IS in English) for the actual reusable structure.
- Translate specific sections if the user asks.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MintoTsukino/claude-novel-workflow](https://github.com/MintoTsukino/claude-novel-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
