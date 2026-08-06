---
trigger: always_on
description: > Local vault overrides: also follow `_CLAUDE.md` for folder conventions and
---

# Obsidian Second Brain - Codex CLI Operating Manual

> Local vault overrides: also follow `_CLAUDE.md` for folder conventions and
> Markdown authoring rules. In particular, `._trash&cache/` is reserved for
> temporary stitching/cache files and soft trash, and Mermaid diagrams in new
> Markdown should default to the `forest` theme with compact, whole-graph
> layouts.

This vault runs the **obsidian-second-brain** skill. The skill ships a set of
*commands*: each one is a multi-step instruction file that you (the Codex
agent) should follow when the user's request matches its trigger phrase.

## How to operate

1. Read `_CLAUDE.md` in the vault root, if it exists, to learn the user's
   vault conventions (folder map, daily note format, naming).
2. When the user's request matches a trigger in the tables below, read the
   matching file under `.codex/commands/<name>.md` and follow its
   instructions step by step.
3. Treat the AI-first vault rule (`.codex/references/ai-first-rules.md`) as
   non-negotiable for every note you write: `## For future Claude` preamble,
   rich frontmatter (`type`, `date`, `tags`, `ai-first: true`), `[[wikilinks]]`
   for every person/project/concept, recency markers per external claim,
   sources verbatim, confidence levels where applicable.
4. Do not invent commands. If no command matches, ask the user what they
   want or fall back to plain natural-language help.

## Command routing tables (grouped by category)

### Wiki layer - simplified knowledge operations

| Command | What it does | Read this file |
|---|---|---|
| `/ingest` | LLM_wiki intake protocol - preserve raw sources, extract learning units, and update durable wiki knowledge | `02.wiki/projects/LLM_wiki Wiki Layer Protocol.md`, then `.codex/commands/ingest.md` |
| `/query` | LLM_wiki retrieval protocol - search, explain, synthesize, and answer from vault evidence | `02.wiki/projects/LLM_wiki Wiki Layer Protocol.md`, then `.codex/commands/query.md` |
| `/lint` | LLM_wiki maintenance protocol - audit links, duplicates, stale claims, raw backlog, DailyNotes drift, Mermaid defaults, and Mentor consistency | `02.wiki/projects/LLM_wiki Wiki Layer Protocol.md`, then `.codex/commands/lint.md` |

### Vault - daily writing, capture, find

| Command | What it does | Read this file |
|---|---|---|
| `/obsidian-board` | Show or update a kanban board - flags overdue items, updates from conversation | `.codex/commands/obsidian-board.md` |
| `/obsidian-capture` | Quick idea capture - zero friction, saves to the maintained capture area and mentions in daily note | `.codex/commands/obsidian-capture.md` |
| `/obsidian-daily` | Create or update today's daily note - pulls calendar events, overdue tasks, and conversation context | `.codex/commands/obsidian-daily.md` |
| `/obsidian-find` | Smart vault search - returns results with context, not just filenames | `.codex/commands/obsidian-find.md` |
| `/obsidian-log` | Log this work or dev session to the vault - infers project from context | `.codex/commands/obsidian-log.md` |
| `/obsidian-person` | Create or update a person note from conversation context | `.codex/commands/obsidian-person.md` |
| `/obsidian-project` | Create or update a project note - adds to board and daily note automatically | `.codex/commands/obsidian-project.md` |
| `/obsidian-projects` | Live project status from git + local docs - infers all context from vault notes, no config required | `.codex/commands/obsidian-projects.md` |
| `/obsidian-recap` | Summarize a time period from the vault - today, week, or month | `.codex/commands/obsidian-recap.md` |
| `/obsidian-recurring` | Track a recurring obligation (payment, filing, ops) with a cadence and a computed next-due date | `.codex/commands/obsidian-recurring.md` |
| `/obsidian-save` | Save everything worth keeping from this conversation to the vault | `.codex/commands/obsidian-save.md` |
| `/obsidian-task` | Add a task to the right kanban board with inferred priority and due date | `.codex/commands/obsidian-task.md` |
| `/obsidian-world` | Load your identity, values, priorities, and current state in one shot - with progressive context levels to avoid burning tokens | `.codex/commands/obsidian-world.md` |

### Thinking - synthesis, decisions, learning, reviews

| Command | What it does | Read this file |
|---|---|---|
| `/idea-discovery` | Surface 3-5 next-direction candidates by reading ungraduated ideas, open project questions, and orphan research notes - what is worth working on next | `.codex/commands/idea-discovery.md` |
| `/obsidian-adr` | Generate a decision record when the vault structure changes - the vault knows why it knows what it does | `.codex/commands/obsidian-adr.md` |
| `/obsidian-challenge` | Red-team your current idea against your own vault history - finds contradictions, past failures, and flawed assumptions | `.codex/commands/obsidian-challenge.md` |
| `/obsidian-connect` | Bridge two unrelated domains using your vault's link graph - forces creative friction to spark new ideas | `.codex/commands/obsidian-connect.md` |
| `/obsidian-decide` | Extract decisions from this conversation and log them to the right project notes | `.codex/commands/obsidian-decide.md` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VectorPeak/LLM-Wiki](https://github.com/VectorPeak/LLM-Wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
