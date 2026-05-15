---
trigger: always_on
description: This file is the authoritative schema for this wiki. Any LLM agent maintaining this wiki must read this file first and follow its rules exactly.
---

# CLAUDE.md — Personal Wiki Schema

This file is the authoritative schema for this wiki. Any LLM agent maintaining this wiki must read this file first and follow its rules exactly.

---

## 1. Project Overview

This is a personal knowledge wiki built and maintained by LLMs. It follows a three-layer architecture: raw sources (immutable input entries the LLM reads but never writes), the wiki (a living knowledge base the LLM owns completely), and this schema file (the rules governing how the wiki is built and maintained). The wiki captures one person's life in encyclopedic form — capturing places, people, ideas, decisions, patterns, and eras as they appear across journals, notes, and other personal sources. The project is inspired by Karpathy's LLM Wiki pattern: the LLM acts as a tireless editor who reads raw entries and synthesizes them into a coherent, cross-linked knowledge base, not a diary, not a log, but a wiki.

---

## 2. Directory Conventions

```
raw/       — immutable source entries. LLM reads, never writes.
wiki/      — LLM-generated knowledge base. LLM owns completely.
scripts/   — ingestion scripts. Human and LLM maintain together.
```

- `raw/` contains subdirectories per source (e.g., `raw/obsidian/`). Each entry is a single file. Filenames encode date and ID. The LLM reads these to absorb content into the wiki.
- `wiki/` contains all article files plus `index.md` and `log.md`. The LLM creates, edits, and reorganizes files here freely.
- `scripts/` contains Python ingestion scripts that convert source data into raw entries. The LLM may read and modify scripts but does not run them autonomously.

Do not create any other top-level directories without updating this schema.

---

## 3. Data Source Config

```yaml
sources:
  obsidian:
    path: ~/path/to/your/obsidian-vault/
    script: scripts/ingest_obsidian.py
  apple_notes:
    path: (macOS Notes app via AppleScript)
    script: scripts/ingest_apple_notes.py
  documents:
    path: ~/path/to/your/documents/
    script: scripts/ingest_documents.py
  # notion: (future)
```

The ingestion scripts read source files and write structured entries into `raw/<source>/`. Each entry file follows this naming convention: `YYYY-MM-DD_<id>.md`. Entries are append-only once written. The LLM never modifies files in `raw/`.

---

## 4. Taxonomy

Directories inside `wiki/` are organized by type. The LLM creates directories as needed when articles of a new type are first written.

**Rule: Directories emerge from data. Never pre-create empty directories.**

| Directory | Type Label | What Goes Here |
|---|---|---|
| `wiki/people/` | person | Named individuals who appear in the subject's life |
| `wiki/projects/` | project | Discrete efforts the subject worked on or contributed to |
| `wiki/places/` | place | Physical locations with personal significance |
| `wiki/events/` | event | Specific occurrences: conferences, encounters, ceremonies, incidents |
| `wiki/companies/` | company | Organizations the subject worked at, worked with, or was shaped by |
| `wiki/institutions/` | institution | Universities, governments, nonprofits, collectives |
| `wiki/books/` | book | Books read or encountered with meaningful impact |
| `wiki/films/` | film | Films watched that left a mark |
| `wiki/music/` | music | Artists, albums, or songs with personal resonance |
| `wiki/games/` | game | Video games, board games, or play experiences |
| `wiki/tools/` | tool | Software, apps, or hardware used repeatedly |
| `wiki/platforms/` | platform | Platforms the subject used, built on, or studied |
| `wiki/courses/` | course | Formal or informal learning experiences |
| `wiki/publications/` | publication | Newsletters, journals, blogs, or media outlets |
| `wiki/philosophies/` | philosophy | Named belief systems, frameworks, or worldviews encountered |
| `wiki/patterns/` | pattern | Recurring behaviors, tendencies, or habits in the subject's life |
| `wiki/tensions/` | tension | Persistent internal conflicts or unresolved contradictions |
| `wiki/identities/` | identity | Self-conceptualizations the subject held at different points |
| `wiki/life/` | life | Overview articles about periods or domains of the subject's life |
| `wiki/eras/` | era | Multi-year life phases with a coherent character |
| `wiki/transitions/` | transition | Pivot moments between eras or life configurations |
| `wiki/decisions/` | decision | Specific choices made, with context and consequence |
| `wiki/experiments/` | experiment | Deliberate tests: new habits, modes of working, lifestyle changes |
| `wiki/setbacks/` | setback | Failures, losses, and reversals worth understanding |
| `wiki/relationships/` | relationship | Significant dyadic relationships (friendship, romance, partnership) |
| `wiki/mentorships/` | mentorship | Relationships structured around learning or guidance |
| `wiki/communities/` | community | Groups the subject belonged to or was shaped by |
| `wiki/strategies/` | strategy | High-level approaches the subject used to navigate life or work |
| `wiki/techniques/` | technique | Specific repeatable methods or practices |
| `wiki/skills/` | skill | Capabilities developed over time |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cylqwe7855-alt/personal-wiki](https://github.com/cylqwe7855-alt/personal-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
