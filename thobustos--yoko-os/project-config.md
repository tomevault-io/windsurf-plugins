---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Personal Agent OS - Claude Instructions

This is the core operating system for a personal life management vault. Claude should behave as an intelligent assistant that understands the vault structure, maintains information freshness, and helps the user live intentionally.

---

## CRITICAL: First Action on Every Session

**Before doing ANYTHING else, read the vault path:**

```
Read: config/vault.json
```

This file contains the absolute path to the user's vault. ALL vault operations use this path.

- If `config/vault.json` exists → use the `vault_path` value
- If missing → fall back to `../my-vault/` relative to this repo
- NEVER hardcode vault paths in skills or instructions

This is the foundation of OpenYoko - the vault is where all personal data lives.

---

## Repository Overview

This repo (`personal-agent-os/`) is the **framework and templates**. The user's actual vault lives at `../my-vault/` as a sibling directory.

**Key directories:**
- `.claude/skills/` - Skill definitions (SKILL.md files) that define workflows
- `templates/` - Markdown templates for vault entries
- `docs/` - User documentation
- `example-vault/` - Reference vault structure
- `config/mcp/` - MCP server configuration (credentials stored outside repo)

---

## Philosophy

> Structure before automation. Clarity before tools.

Personal Agent OS organizes life around:
- **Identity** - Who you are becoming (LIFE_VISION.md)
- **Pillars** - What must never be neglected (PILLARS.md)
- **Projects** - What you're building (03_PROJECTS/)
- **Relationships** - Who matters (04_PEOPLE/)
- **Reflection** - How you course-correct (02_JOURNAL/)

The vault is the single source of truth. Claude's job is to:
1. Keep the vault accurate and fresh
2. Surface relevant context when helping
3. Capture new information in the right place
4. Maintain the integrity of the system

---

## Information Architecture

### Vault Location

**For file operations, use the absolute vault path from `config/vault.json`.**

Setup:
1. Copy `config/vault.example.json` → `config/vault.json`
2. Set your vault's absolute path (e.g., `/Users/yourname/Documents/my-vault`)

On session start: Read `config/vault.json` to get the vault path. If it doesn't exist, fall back to resolving `../my-vault/` from the repo root.

### Folder Structure
```
my-vault/
├── 00_SYSTEM/           # Operating system - vision, pillars, state
│   ├── GLOBAL_STATE.md  # Current focus, energy, active projects
│   ├── LIFE_VISION.md   # 5-year vision, identity word
│   ├── PILLARS.md       # 10 life pillars with questions
│   ├── IMPORTANT_DATES.md # Recurring dates, rituals, anniversaries
│   ├── TODO.md          # Master cross-project to-do list
│   ├── ASSETS.md        # Digital assets: domains, subscriptions, accounts
│   ├── extensions/      # Skill extensions (vault-specific customizations)
│   └── OPS/             # System operations and rituals
│       ├── activity-logs/  # Weekly session activity logs
│       ├── scans/       # Pulse logs and deep scan reports
│       └── granola-inbox/  # Raw meeting transcripts synced by the Granola Obsidian plugin (flat, dated files, attendees in frontmatter)
├── 01_GOALS/            # Life → Year → Quarter cascade
├── 02_JOURNAL/          # Weekly/ and Monthly/ reflections
├── 03_PROJECTS/         # Active projects with _STATE.md each
├── 04_PEOPLE/           # Relationship notes
├── 05_WRITING/          # Drafts/, Published/, Ideas/, Reflections/
├── 06_READING/          # Papers/, Daily/, Highlights/, Synthesis/
└── 07_ARCHIVE/          # Completed projects, historical context
```

### Key Files to Read
1. **GLOBAL_STATE.md** - Always read first. Contains current focus, energy, active projects, and **default integrations** (Google, GitHub accounts).
2. **IMPORTANT_DATES.md** - Recurring dates, rituals, relationship maintenance. Read during scans to surface upcoming dates.
3. **Project _STATE.md** - Read before discussing any specific project.
4. **_GUIDE.md** files - Each folder has operating instructions.

### Meeting Transcripts

Raw meeting transcripts sync automatically into `00_SYSTEM/OPS/granola-inbox/` via the Granola Obsidian plugin (no LLM step in the sync). Each file carries `title` and `attendees` in frontmatter, so meetings can be skimmed or queried directly (Obsidian search, Dataview) without opening every file. Nothing routes these into project folders automatically. `/daily` only reads them for reflection context; it does not move or classify them.

---

## Skills Available

Skills are invoked with `/skill-name` or by trigger phrases. All skills are in `.claude/skills/`.

**Private skills** (underscore-prefixed) live in the vault at `{{vault_path}}/00_SYSTEM/skills/` and are not committed to this repo. Load them from there.

### Cadence Skills (Run Regularly)
| Skill | When | Purpose |
|-------|------|---------|
| `/onboarding` | First time | Create vault, gather vision |
| `/daily` | Morning/evening | Set intention, log reflection |
| `/weekly` | Sunday | Close week, plan next |
| `/monthly` | 1st of month | Deep reflection, pillar scores |

### Project Skills

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ThoBustos/yoko-os](https://github.com/ThoBustos/yoko-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
