---
trigger: always_on
description: You are working inside an Obsidian vault that runs one person's life: journal, planning, habits, tasks, people, writing, reading. Everything here is personal data. Whatever you read in a chat is sent to a model provider, so read only what the current job needs, never copy journal text into other notes or outside the vault, and never invent entries.
---

# Compass vault: instructions for AI agents

You are working inside an Obsidian vault that runs one person's life: journal, planning, habits, tasks, people, writing, reading. Everything here is personal data. Whatever you read in a chat is sent to a model provider, so read only what the current job needs, never copy journal text into other notes or outside the vault, and never invent entries.

You have judgment, not authority. Analyse, summarise, draft, and recommend freely. Every change to a file is proposed first and applied only after the person says yes.

## Folder map and what you may do there
| Folder | What it is | You may |
| --- | --- | --- |
| `00 Dashboards/` | DataviewJS dashboards generated from properties; `Setup.md` is the onboarding page | read; edit only if asked to change a dashboard |
| `01 Journal/Daily`, `Weekly`, `Quarterly` | periodic notes named `YYYY-MM-DD`, `gggg-Www`, `YYYY-QN` | read; append under an existing `##` heading when asked |
| `02 Retreats/` | `YYYY-QN Personal Retreat` notes with `wheel_*` scores | read; fill sections with the person's own words when asked |
| `03 Planning/` | Life Theme, Core Values (+ roles), Ideal Week | read; edit only on explicit request, section by section |
| `04 Projects/` | project notes (`#project/<slug>` tasks) and `Projects Board.md` (Kanban) | read and edit on request |
| `05 People/` | people notes (`#p/<slug>` tasks, `#discuss` roll-ups) | read and edit on request |
| `06 Writing/` | one folder per type, each with a Kanban board | read and edit on request; this is where drafting help happens |
| `07 Library/Book Notes` | book notes with `^block-id` quotes | read and edit on request |
| `08 Tasks/Tasks.md` | master task list; captured to, never read by hand | append tasks under `## Inbox` when asked |
| `09 Reading/` | reading plan, chapter, verse, study, and topic notes (Bible is the worked example) | read |
| `Prompts/` | the prompt library: one note per recurring job | read; follow the note's `## Prompt` section when asked to run it |
| `Templates/` | Templater templates; property lists come from `Meta/Compass Config.md` | edit only when asked to change the system |
| `Meta/Compass Config.md`, `Meta/views/*.js` | the single config (questions, habits, wheel areas, folders) and dashboard widgets | read; edit `Compass Config.md` only when the person asks to change their questions, habits, areas, or birthdate |
| `Guide/` | how the system works and why | read first when unsure |
| `wiki/`, `inbox/` | knowledge layer (claude-obsidian plugin, optional) | follow `wiki/routing-map.md`; writes go through the plugin's inspect, approve, apply transaction |
| `scripts/` | maintainer tools (reading plan generator, template build) | read |
| `.obsidian/` | app and plugin settings | never edit |

## Conventions
- Daily questions are `dq_*` number properties (1 to 10, effort not results). Habits are `habit_*` checkbox properties. Wheel of life is `wheel_*` in retreat notes. The lists live in `Meta/Compass Config.md` (`questions`, `habits`, `wheel_areas`); dashboards discover them by prefix. Never rename or remove keys in existing notes.
- Tasks use the Obsidian Tasks emoji format: `- [ ] text 📅 YYYY-MM-DD` due, `⏳` scheduled, `🔁` recurring, `⏫` high priority, `➕` created. Routing tags: `#project/<slug>`, `#p/<slug>`, `#discuss`. Slug = note title lowercased, non-alphanumerics to `-`; Project and Person notes print their tag at the top.
- Links are `[[wikilinks]]`. Dates are ISO in file names and properties. No em dashes anywhere.
- Notes tagged `example` are seed data. Do not treat them as the person's real life; offer to delete them once real entries exist.
- Prefer appending under an existing heading to creating notes. New notes go in the folder whose Templater folder template fits: create them empty at the right path, let Templater fill them, then patch.
- When reviewing a week or a quarter, read the daily notes first and quote the person's own words back. Summarise, do not grade.

## How to find things
- Today: `01 Journal/Daily/<today>.md`. This week: `01 Journal/Weekly/<gggg-Www>.md`. This quarter and retreat: `01 Journal/Quarterly/<YYYY-QN>.md`, `02 Retreats/<YYYY-QN> Personal Retreat.md`.
- Tasks: `00 Dashboards/Task Dashboard.md` explains the queries; the data is in `08 Tasks/Tasks.md`, `04 Projects/*`, `05 People/*`.
- Boards: any note with `kanban-plugin` in its properties; each `## Heading` is a lane, each `- [ ]` line a card.
- System questions: `Guide/00 Start Here.md`, then the workflow guide it points to. Setup state: `00 Dashboards/Setup.md`.
- Knowledge layer routing: `wiki/routing-map.md`.

## Driving Obsidian (MCP server `obsidian`)
Prefer these tools over raw file access when they are available; they act inside the running app.
- `active_file_get_path` first whenever the person says "this note".
- Read: `vault_read`, `vault_get_document_map` (one section), `vault_list`, `search_simple`, `search_query`, `tag_list`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AgriciDaniel/compass](https://github.com/AgriciDaniel/compass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
