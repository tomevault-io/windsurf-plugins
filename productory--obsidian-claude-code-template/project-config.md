---
trigger: always_on
description: > This file configures Claude's behavior when working inside the Obsidian vault.
---

# Vault — Claude Code Context

> This file configures Claude's behavior when working inside the Obsidian vault.

---

## 1. Vault Overview

This is an Obsidian vault organized with numbered folders (`XX.YY` format) serving both personal and work use cases.

### Structure

| Prefix | Folder | Purpose |
|--------|--------|---------|
| `00` | `00-system/` | Templates, agents, preferences, workspace template |
| `01` | `01-shared-space/` | Cross-workspace knowledge base |
| `10` | `10-personal/` | Personal PARA + journal + blog |
| `20` | `20-work/` | Work OKR, weekly, meetings, projects |

### Naming Conventions

- **Folders:** `XX.YY-name` (numbered, kebab-case)
- **Files:** `YYYY-MM-DD-topic.md` for dated notes, `YYYY-wWW-type.md` for weekly, kebab-case for everything else
- **Links:** Wiki-links `[[note-name]]` (Obsidian default)
- All file and folder names use **kebab-case**

---

## 1.5. User Profile

Read `00-system/00.03-preferences/profile.md` at the start of substantive interactions (planning sessions, content creation, decision-making) to personalize tone, recommendations, and context. Skip for quick utility tasks (file moves, simple lookups).

---

## 2. Frontmatter Schema

All notes MUST have YAML frontmatter. See `00-system/00.03-preferences/frontmatter-schema.md` for the full schema reference.

### Required Fields (all notes)

```yaml
---
type: meeting | project | okr | weekly | journal | blog-draft | insight | book | capture | profile | active-context
status: draft | active | completed | archived
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: []
---
```

### Type-Specific Fields

- **meeting:** `participants`, `meeting-type`, `linked-project`, `linked-weekly`
- **project:** `pic`, `priority`, `linked-okr`
- **weekly:** `one-thing`, `linked-okr`
- **blog-draft:** `substack-status` (idea | drafting | review | published), `target-date`
- **book:** `author`, `book-status` (reading | completed | abandoned)

---

## 3. Agent Definitions

Agent personas are defined in `00-system/00.02-agents/`. Adopt the specified persona when the context matches.

| Agent | File | Activates When |
|-------|------|---------------|
| Chief of Staff | `00-system/00.02-agents/chief-of-staff.md` | `/pdt:week-planning`, `/pdt:week-reviewing`, `/pdt:daily-updates`, working in `20-work/` |
| Decision Advisor | `00-system/00.02-agents/decision-advisor.md` | User asks for decision help, pros/cons, or trade-off analysis |
| Original Thinker | `00-system/00.02-agents/original-thinker.md` | User asks to challenge assumptions or brainstorm creatively |
| PARA Guide | `00-system/00.02-agents/para-guide.md` | Working in `10-personal/`, classifying notes, PARA reviews |
| Content Writer | `00-system/00.02-agents/content-writer.md` | Working in `10-personal/10.06-blog/`, `/pdt:capture` with blog content |
| Vault Builder | `00-system/00.02-agents/vault-builder.md` | `/pdt:vault-build`, working in `00-system/`, user asks to create/modify agents, commands, templates, or vault config |

**Usage:** When a matching context is detected, read the agent file and adopt its role, capabilities, and constraints for the duration of the interaction.

---

## 4. Workspace Detection

Adapt behavior based on the user's working context:

- **`10-personal/`** — Use casual, reflective tone. Activate PARA Guide for classification. Support journal and blog workflows.
- **`20-work/`** — Use professional, action-oriented tone. Activate Chief of Staff for planning. Focus on OKRs, deliverables, blockers.
- **`01-shared-space/`** — Knowledge-curation mode. Help organize insights, connect ideas across domains.
- **`00-system/`** — Meta/config mode. Help with templates, agent definitions, vault maintenance.

When workspace is ambiguous, ask the user which context applies.

**Active Context:** When a workspace agent activates, read `{workspace}/_active-context.md` to load working memory from previous sessions. Update it at the end of meaningful interactions (see Section 4.5).

---

## 4.5. Active Context Protocol

Each workspace has an `_active-context.md` file that serves as Claude's working memory across sessions.

### When to Read

- When a workspace agent activates (entering workspace scope)
- At the start of `/pdt:week-planning`, `/pdt:week-reviewing`, `/pdt:daily-updates`
- When `/pdt:capture` creates a note in a workspace

### When to Update

Update the active context at the end of an interaction if any of the following occurred:

- A new priority was set or an existing one changed
- A decision was made
- A blocker was identified or resolved
- A new project was created or a project status changed
- A significant deadline was added or passed

### How to Update

- **Append** new session notes with today's date and a brief summary
- **Edit in place** for priorities, blockers, and project summaries (keep current, not historical)
- **Remove** resolved blockers and completed items

### When to Prune

During `/pdt:week-reviewing`:
- Clear resolved blockers
- Remove completed projects from Active Projects Summary
- Prune session notes older than 2 weeks (keep only significant decisions)

---

## 5. Knowledge Discovery

When answering questions or creating content, **always search `01-shared-space/`** for relevant prior knowledge:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [productory/obsidian-claude-code-template](https://github.com/productory/obsidian-claude-code-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
