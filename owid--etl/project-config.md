---
trigger: always_on
description: Our World in Data's ETL system - a content-addressable data pipeline with DAG-based execution.
---

# Agent Guide

Our World in Data's ETL system - a content-addressable data pipeline with DAG-based execution.

## Critical Rules

- **Always use `.venv/bin/`** for all Python commands (`etl`, `python`, `pytest`)
- **Never mask problems** - no empty tables, no commented-out code, no silent exceptions
- **Trace issues upstream**: snapshot → meadow → garden → grapher
- **`dag/archive/*.yml` is a generated record** — it is reconstructed from git history by `etl archive-dag`, so never hand-edit it. It lists steps that were once active (with the commit where they were last active) purely for recovery; to bring one back, `git checkout` that commit.
- **Never delete a step without archiving it.** Removing or superseding an active step (new version, retirement, replacement) obligates you to archive it — deleting the files alone is a bug. Procedure: remove its `dag/*.yml` entry and delete its files → **commit** → run `etl archive-dag` (it reads *committed* history, so the removal must be committed first) → commit the regenerated `dag/archive/*.yml`. If `archive-dag` sweeps in unrelated steps others left un-archived, `git checkout` those files to keep your PR scoped (never hand-edit the archive). For a migrated/backport dataset, also delete its now-orphaned `snapshots/backport/latest/dataset_<id>_*` mirror files.
- **Ask the user** if unsure - don't guess
- **Always run `make check` before committing**
- If not told otherwise, save outputs to `ai/` directory.
- **Notebooks**: Always create AND execute immediately using `uv run jupyter nbconvert --to notebook --execute --inplace <path>`
- **Skills**: When creating new skills in `.claude/skills/`, always include `metadata: { internal: true }` in the SKILL.md frontmatter unless the user explicitly asks for the skill to be public. This prevents external skill indexes from crawling and listing our internal skills.

## Team

Everything you post to GitHub or Slack goes out under a **human's identity**. Any text you author and post that a reader could take for the human's own words **must** carry the attribution line below. This is mandatory — not a judgment call about whether the comment is "worth it."

1. **Attribute the work.** Put this blockquote as the *first line* of the content:

   ```
   > _Written by Claude <model name> — @<handle> at the wheel._
   ```

   Replace `<model name>` with the human-readable name of the model actually generating the content (e.g. "Sonnet 5", "Opus 4.8", "Fable 5", "Haiku 4.5") — not the literal string "Code". Keeping the "Claude" prefix makes the attribution recognizable even to readers unfamiliar with individual model names.

   It applies to **every** surface, **every** time you post:
   - PR descriptions / bodies
   - PR issue-level comments
   - **Inline review comments _and_ replies to review comments** (e.g. answering Codex / Copilot / a reviewer)
   - Standalone Slack messages or drafts

   Use the handle of the human directing the work (usually the current git user; ask if ambiguous).

   **The only exception** is a comment that is a bare mechanical token with *no prose* — a lone `@codex review` ping or a 👍. The moment your comment contains a sentence of explanation, it needs the line. When in doubt, include it.

2. **Use exact handles** from the table below when tagging colleagues. Don't guess — a wrong tag pings a real person. If a name isn't in this table, write the plain name (e.g. "Bastian") instead of `@`-tagging, and ask the user for the handle.

   | Name | GitHub handle |
   |---|---|
   | Pablo A Rosado | `@pabloarosado` |
   | Pablo Arriagada | `@paarriagadap` |
   | Veronika Samborska | `@veronikasamborska1994` |
   | Mojmir Vinkler | `@Marigold` |
   | Lucas Rodés-Guirao | `@lucasrodes` |
   | Tuna Acisu | `@antea04` |
   | Fiona Spooner | `@spoonerf` |
   | Edouard Mathieu | `@edomt` |

   The disclosure rule does **not** apply to OWID-reader-facing artifacts (e.g. the `/latest` data-update post on ourworldindata.org) — those are authored by the named human, not by Claude.

3. **This repo is public — keep internal context out of it.** PR descriptions, commit messages, and issue/review comments must never identify people who contact us (no names, roles, or employers — say "a reader pointed out ..." instead), and must not reference internal discussions (Slack threads, Notion docs) or who suggested what internally. Motivate changes using public facts only; internal context stays internal.

## Pipeline Overview

**snapshot** → **meadow** → **garden** → **grapher** → **export**

| Stage | Location | Purpose |
|-------|----------|---------|
| snapshot | `snapshots/` | DVC-tracked raw data |
| meadow | `etl/steps/data/meadow/` | Basic cleaning |
| garden | `etl/steps/data/garden/` | Business logic, harmonization |
| grapher | `etl/steps/data/grapher/` | MySQL ingestion |

**Snapshot is raw passthrough only.** It downloads the source files and writes them out using the source's own row labels, column labels, and period labels. That's it. The following all belong in **garden**, not in the snapshot script:

- Summing or merging rows from different source categories into one bucket
- Picking the most recent value when several source files report the same period

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [owid/etl](https://github.com/owid/etl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
