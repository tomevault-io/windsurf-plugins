---
trigger: always_on
description: This is the operating system for an AI-powered product management workspace. Claude Code reads this file at the start of every session so it knows who you are, how your vault is organized, and how to behave.
---

# AI PM Workspace

This is the operating system for an AI-powered product management workspace. Claude Code reads this file at the start of every session so it knows who you are, how your vault is organized, and how to behave.

If you just cloned this repo, run `/personalize` to fill in your identity. Everything below is the template a fresh user sees.

## Identity & Context

<!-- PERSONALIZE: filled automatically by /personalize. -->

**Owner**: [YOUR NAME], [YOUR ROLE] at [YOUR COMPANY]
**Focus areas**: [YOUR PRODUCT AREAS - e.g., "Core Platform, API strategy, customer onboarding"]
**Vault purpose**: Personal PM workspace - private, never referenced externally

**Language**: English
<!-- If you work in multiple languages, specify rules here. Example:
- English: Work content, meetings, decisions, communications
- [Other language]: Personal notes, non-work content
- NEVER mix languages within a note
-->

---

## Company Context

<!-- PERSONALIZE: replaced by /personalize with your actual context. -->

**What [YOUR COMPANY] does**: [Brief description of what your company does - 1-2 sentences]

**Target customers**: [Who your customers are - e.g., "Enterprise SaaS companies", "SMB retailers"]

**Core products**:
- **[Product 1]**: [Brief description]
- **[Product 2]**: [Brief description]

**Target users**:
- **[User type 1]**: [Brief description]
- **[User type 2]**: [Brief description]

---

## How tasks live in this repo

Tasks live in [Dashboard/tasks.md](Dashboard/tasks.md) - a plain-markdown checklist with three sections:

- **This week** - what you've committed to.
- **Next** - intended soon, not this week.
- **Backlog** - unscheduled.

Skills write to `Dashboard/tasks.md` by default (no external task manager required). Each task line looks like:

```
- [ ] Action description - source: [[note title]] - due: YYYY-MM-DD - priority: P3
```

Priority, due date, and source are all optional.

**Optional Todoist sync**: If you prefer Todoist, follow `docs/todoist-sync.md` to wire it up. The default template works without it.

---

## MCP Servers Available

<!-- MCPs are configured via `claude mcp add`, not stored in this file. -->
<!-- See setup/mcp-configs/ for setup instructions for each MCP server. -->

### GitHub MCP (recommended)
**Use for**: Repository and issue management.
**Key operations**:
- Check implementation status on epics
- Read engineering comments on issues
- Find related PRs

### [Optional] Documentation MCP
<!-- If your company has a docs MCP (e.g., for public API docs), configure it and note it here. -->

### [Optional] Notion / Linear / Jira
<!-- If you use these tools, configure the respective MCP and note it here. /personalize --deep will help detect what you have. -->

---

## Vault Structure & Conventions

### Folder organization

```
/
|-- Dashboard/             # Living documents
|   |-- tasks.md           # In-repo task list (skills write here)
|   |-- Weekly P-Tasks.md  # Weekly priorities (P1-P5)
|   `-- people-profiles.md # Stakeholder communication profiles
|-- Initiatives/           # Working files for active initiatives (created by /personalize --deep)
|-- epics/                 # Epic drafts (created by /create-epic)
|-- research/              # Competitive research, user journeys, personas
|   |-- personas/          # Persona library (ships with a template + 3 examples)
|   `-- journeys/          # User journeys (created by /user-journey)
|-- journals/              # Daily notes (YYYY/MM-Month/DD-MM-YYYY.md)
|-- Loose Notes/
|   `-- Work/              # Decisions, drafts, analysis
|-- Meetings/              # Meeting notes
|-- samples/               # Sample data used by /guide modules
|-- templates/             # Note templates (daily, meeting, loose, epic, initiative)
|-- docs/                  # Reference docs (epic lifecycle, optional Todoist sync)
`-- CLAUDE.md              # This file
```

### Naming conventions

| Type | Format | Example | Location |
|------|--------|---------|----------|
| Daily note | `DD-MM-YYYY.md` | `14-02-2026.md` | `journals/2026/02-February/` |
| Work note | `YYYY-MM-DD - Title.md` | `2026-02-14 - Decision - API scope.md` | `Loose Notes/Work/` |
| Meeting note | `YYYY-MM-DD - Meeting description.md` | `2026-02-14 - Customer sync.md` | `Meetings/` |
| Initiative file | `[initiative-slug].md` | `billing-migration.md` | `Initiatives/` |
| Epic file | `YYYY-MM-DD - [Title].md` | `2026-04-20 - Unified onboarding dashboard.md` | `epics/` |
| User journey | `YYYY-MM-DD - [Persona] - [Flow].md` | `2026-04-20 - Dana - First API integration.md` | `research/journeys/` |
| Competitive research | `YYYY-MM-DD - Competitive - [Topic].md` | `2026-04-20 - Competitive - Onboarding UX.md` | `research/` |
| Persona | `[persona-name].md` | `example-developer.md` | `research/personas/` |

### P-Tasks system
**Purpose**: Weekly priorities tracked in `Dashboard/Weekly P-Tasks.md`.
**Priority levels**: P1 (critical) down to P5 (optional).
**Status markers** (added during Friday review):
- `✅` done
- `🔄` carried over
- `❌` not done / deprioritized


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aleksander-dytko/ai-pm-workspace](https://github.com/aleksander-dytko/ai-pm-workspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
