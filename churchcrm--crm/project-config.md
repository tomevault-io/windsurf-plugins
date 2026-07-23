---
trigger: always_on
description: This file is loaded by Claude Code for every session. Follow these instructions for all work on this project.
---

# ChurchCRM Development Guide

This file is loaded by Claude Code for every session. Follow these instructions for all work on this project.

---

## Skills System

Structured development skills live in `.agents/skills/`. **Always consult the relevant skill before starting work.**

- **Index**: [`.agents/skills/churchcrm/SKILL.md`](.agents/skills/churchcrm/SKILL.md) — use this to find the right skill for your task
- **Generic skills**: `gh-cli`, `interface-design`, `php-best-practices`, `web-design-guidelines` (see `.agents/skills/`)

### Skill Selection by Task

| Task type | Skills to read |
|-----------|---------------|
| New API endpoint | `api-development.md` → `service-layer.md` → `slim-4-best-practices.md` → `security-best-practices.md` |
| Migrate legacy page | `routing-architecture.md` → `admin-mvc-migration.md` → `frontend-development.md` |
| Database / ORM work | `database-operations.md` → `db-schema-migration.md` |
| UI / frontend changes | `responsive-design-guidelines.md` → `bootstrap-5-migration.md` → `frontend-development.md` → `webpack-typescript.md` |
| Datetime / timezone work | `timezone-handling.md` (event editor, calendar, kiosk, anything cross-tz) |
| i18n / translations | `i18n-localization.md` → `locale-translation-workflow.md` → `frontend-development.md` |
| Security issue | `security-best-practices.md` → `authorization-security.md` |
| New community plugin | `plugin-system.md` → `plugin-development.md` → `plugin-create.md` → `plugin-security-scan.md` |
| Core plugin update (`src/plugins/core/*`) | `plugin-system.md` → `plugin-development.md` → `plugin-migration.md` |
| Admin-side plugin audit | `plugin-system.md` → `plugin-compliance.md` |
| Testing | `testing.md` → `cypress-testing.md` |
| Commit / PR | `git-workflow.md` → `github-interaction.md` |
| Refactor | `refactor.md` → `service-layer.md` |
| Performance | `performance-optimization.md` → `database-operations.md` |
| Configuration | `configuration-management.md` |

---

## Context Optimization

The Claude Code system loads **150+ agent types + 40+ MCP tool schemas** by default (~15-20KB overhead per session). ChurchCRM uses only **8-10 tools** across all workflows.

**Per-Workflow Tool Allowlists** (documented in [`.claude/churchcrm-tools-config.json`](./.claude/churchcrm-tools-config.json)) trim unused tools:
- **Removed:** Google services, email/Slack, Sentry, DataForSEO, all language-specific agents (Rust, Go, Java, etc.), AI media gen, SEO/marketing tools
- **Kept:** GitHub tools (PR/issue ops), Web tools (documentation), Bash (build/git), core code tools, workflow-specific agents
- **Savings:** ~10-13KB per session = **~250K tokens/month** freed for productive work

Workflows automatically load only their required tools (see config file for mapping). No user action needed — this is documentation of what's actually used.

---

## Auto-Learning: Proactive Skill Updates

**IMPORTANT: Agents must update skill files automatically when they learn something new — no user prompt required.**

### When to Update Skills

Update the relevant skill file immediately when you:
- Discover a pattern, API, or convention not yet documented
- Find a bug, gotcha, or anti-pattern worth warning others about
- Solve a recurring problem with a reusable solution
- Confirm that documented guidance is wrong or outdated
- Encounter a new file, class, or service that belongs in the architecture overview

### What Qualifies as "New Learning"

- A class, utility, or helper you had to search for (others will too)
- A constraint you violated and had to fix (e.g., wrong Bootstrap class, missing cast)
- An edge case in Propel ORM, Slim 4, or Tabler/Bootstrap 5 not in existing docs
- A build/test step that's easy to forget
- A new module, route group, or architectural pattern added to the codebase

### What Does NOT Qualify

- Trivialities already covered in existing skill files
- Task-specific context (e.g., "today I fixed issue #1234")
- Speculation — only write confirmed, tested facts
- Anything that duplicates existing documented guidance

### How to Update

1. **Identify the right skill file** from `.agents/skills/churchcrm/SKILL.md`
2. **Edit the skill file** — add a clearly labelled subsection with a short explanation + code example
3. **If it's a new category**, add a row to the table in `.agents/skills/churchcrm/SKILL.md`
4. **Keep it concise** — one paragraph max, prefer code examples over prose
5. **Date the entry** — append `<!-- learned: YYYY-MM-DD -->` as an HTML comment on the section header line

### Memory File Sync

After updating a skill file, also check if [`.claude/projects/.../memory/MEMORY.md`] needs a one-line summary added under **Critical Patterns**.

---

## After PR Review Sessions

- After completing PR review fixes and pushing, always update the relevant skill files in `.claude/skills/` with new learnings (cypress-testing.md, api-development.md, git-workflow.md, etc.) before ending the session
- If no genuine new learnings emerged, explicitly say so rather than padding with trivia

---

## Always-Apply Standards

These rules apply to **every code change** in this project.

@.agents/skills/churchcrm/code-standards.md

---

## Git & PR Workflow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChurchCRM/CRM](https://github.com/ChurchCRM/CRM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
