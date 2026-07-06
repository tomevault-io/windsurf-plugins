---
trigger: always_on
description: This file provides context about this Obsidian vault for Claude Code.
---

# CLAUDE.md

This file provides context about this Obsidian vault for Claude Code.

> **First time here?** Run `/setup` to personalize this vault — it takes about 2 minutes and fills in all the placeholders below.

---

## Session Startup

**MANDATORY: Read these files immediately at the start of every session, before doing anything else. Do not skip, even in short sessions.**

1. `04_RESOURCES/Context/company-context.md` — Company, customers, market position
2. `04_RESOURCES/Context/product-context.md` — Product capabilities, architecture, value proposition
3. `04_RESOURCES/Context/process-context.md` — Team process, tools, rhythms
4. `04_RESOURCES/MEETINGS.md` — All recurring meetings: participants, purpose, agenda, vault paths

This context is essential for all work in this vault. Do not wait to be asked. Do not skip based on session length or topic.

---

## Vault Context

**Owner**: [YOUR NAME] — [YOUR ROLE]
**Primary Focus**: Professional knowledge management, work tracking, and AI-assisted decision-making
**Key Work Activities**: [e.g., Management, Strategy, Product, Engineering, Sales — customize this]
**Product / System**: [YOUR PRODUCT OR SYSTEM — e.g., a software platform, internal tool, or domain you manage]
**Vault Scope**: Professional work only
**Language Note**: [YOUR PREFERRED LANGUAGE — e.g., US English / German / both]

---

## Company & Team Structure

**Company**: [YOUR COMPANY NAME]
**My Role**: [YOUR ROLE] reporting to [YOUR MANAGER NAME]

**Key Contacts:**
- **[YOUR MANAGER NAME]** — [their role]
- **[TEAM LEAD / ENGINEERING MANAGER NAME]**: Key collaborator for technical scoping and team alignment.
- *Add others as needed*

**Direct Reports** *(if applicable — used by meeting and review skills)*:
- [Name 1] — [Role]
- [Name 2] — [Role]
- *Add or remove as needed*

---

## Organization

The vault uses a 6-folder structure:
- **00_INBOX**: Entry point for all new notes and captures.
- **01_ACTIVE**: Operational layer — Daily Notes, FOCUS, and IDEAS live here.
  - **Daily Notes/**: Current daily notes before archiving.
- **02_PROJECTS**: Active projects with a deadline and specific goal.
- **03_AREAS**: Ongoing responsibilities without deadlines.
  - **Organization/**: Recurring meetings and organizational formats, each in its own thematic subfolder. **NEVER save meeting notes directly in `03_AREAS/Meetings/` — always use a thematic subfolder.**
  - **People/**: Notes organized by relationship:
    - `People/Team/[Name]/` — 1:1s and notes for direct reports
    - `People/Peers/` — Notes for peer colleagues
    - `People/Leadership/` — Leadership contacts
  - **Learning/**: Ongoing learning and reading.
- **04_RESOURCES**: Reference materials, templates, and context files.
  - **Templates/**: Note templates (Daily Note, Meeting Preparation, Manager 1on1, Weekly Note)
  - **Context/**: Company, product, and process context files
  - **Weekly Reviews/**: Weekly synthesis notes
- **05_ATTACHEMENTS**: Storage for images, PDFs, and other binary files.
- **06_ARCHIVE**: Completed projects and inactive items.
  - **Daily Log**: Archived daily notes organized by `YYYY/MM_Month`

---

## Obsidian Vault Conventions

- Always read existing templates and frontmatter patterns in the vault BEFORE creating or editing notes. Match the existing tag names, template structure, and folder hierarchy exactly.
- Structure: Projects (02_PROJECTS) = time-bound deliverables, Areas (03_AREAS) = ongoing responsibilities, Resources (04_RESOURCES) = reference material. When unsure, check existing folder contents.
- **Meeting File Placement**: NEVER save a meeting note directly in `03_AREAS/Meetings/`. Always place it in `03_AREAS/Organization/[Thematic Subfolder]/`. Check existing subfolders first; create a new one only if no suitable folder exists.

---

## Project Structure Conventions

Every project in `02_PROJECTS/` follows a standard structure. Full reference: [[project-structure-conventions]].

**Standard folders:** `context/` · `resources/` · `output/` · `archive/` · `iterations/`

**Standard files:**
- `CLAUDE.md` — project context for Claude (goal, role, key files, working principles, session summary)
- `PLAN.md` — execution plan with phases, open decisions, next steps (version-stamped)
- `README.md` — human-readable overview

**`iterations/`** — one file per session (`iteration-01.md`, `iteration-02.md`, ...) for session continuity. Claude reads the latest iteration to restore context at session start.

---

## User Preferences

- **Tone**: Casual and compact. No fluff — text must be readable and efficient.
- **Language/Dialect**: [YOUR LANGUAGES — e.g., English / German / both]
- **Daily Notes**: Stored in `01_ACTIVE/Daily Notes`, archived monthly to `06_ARCHIVE/Daily Log/YYYY/MM_Month`.

---

## Custom Instructions

- **Memory Maintenance**: When updating `CLAUDE.md`, NEVER append an "Updates" section with a date. ALWAYS rewrite the relevant sections to integrate the new information seamlessly. The file must remain a clean, unified source of truth.
- **Linking Strategy**: Use `[[WikiLinks]]` for all internal references to ensure graph connectivity. **CRITICAL**: NEVER wrap `[[WikiLinks]]` in backticks or code blocks.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [timominkus/obsidian-para-gtd-ai](https://github.com/timominkus/obsidian-para-gtd-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
