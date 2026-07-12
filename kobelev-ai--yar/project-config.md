---
trigger: always_on
description: > **First time?** Run `/setup` to personalize this workspace.
---

# Yar — AI Operating System for Executives

> **First time?** Run `/setup` to personalize this workspace.
> After setup, this section will contain your profile. Edit freely to customize.

## Owner Profile

- **Name:** (run /setup to configure)
- **Role:** (run /setup to configure)
- **Company:** (run /setup to configure)
- **Key focus areas:** (run /setup to configure)

---

## Session Start

> At the start of every session, run `session-starter` agent for a proactive briefing.

The agent checks:
- Date/time and day of week
- Overdue promises and deadlines
- Today's meetings and upcoming days
- Tasks needing attention
- Weekly goals progress

---

## Structure

```
inbox/                        # Global inbox — drop ANYTHING here
.yar/                         # System folder (Yar state — don't edit manually)
  version.md                  # Yar distribution version + update history
  installed.md                # Registry of installed packages
  packages/                   # Archive of installed recipe files
  migrations/                 # Applied migrations (for future updates)
context/
  me.md                       # Owner profile
  me/
    decisions.md              # Decisions (immutable, append-only)
    preferences.md            # Preferences (updateable)
    speaker_signals.md        # Speech patterns for transcript identification
    boundaries.md             # Confidentiality rules (owner-defined only)
    ideas.md                  # Ideas & insights log (append-only)
  projects/                   # Active projects with context
  people/                     # Key people cards
  meetings/                   # Meeting transcripts
    processed/                # Processed transcripts
    docs/                     # Meeting-related documents
    index.json                # Meeting search index
    speaker_mappings.json     # Speaker identification results
  goals/                      # Goals (yearly, quarterly, weekly)
  journal/                    # Daily journal
tasks/
  todo.md                     # Task list (live document)
  archive/                    # Weekly archives
```

### `context/` vs `.yar/` — separation of concerns

- **`context/`** = the owner's content (personality, projects, meetings, goals). User-facing, meaningful to read.
- **`.yar/`** = system state of this Yar instance (what's installed, version, migrations). Service files, usually not read by the owner directly.
- Parallel to `.claude/` (Claude Code config) — dotfolder convention for "don't touch unless you know what you're doing".

---

## Inbox — Single Entry Point

**Everything goes into `inbox/`.** Transcripts, documents, notes, screenshots, recipe files, any file.

The owner drops files into `inbox/` and says "process inbox" or `/inbox`. The system:

1. Reads each file in `inbox/`
2. Determines the type: Yar recipe, meeting transcript, task list, project document, personal note, etc.
3. Calls the appropriate agent/skill:
   - **Yar recipe (`.md` with frontmatter `type: yar-recipe`) → `/pkg_installer`** → guides owner through setup, records in `.yar/installed.md`
   - Meeting transcript → `meeting-processor` → result in `context/meetings/processed/`
   - Tasks/ideas → `todo-processor` → result in `tasks/todo.md`
   - Project info → update `context/projects/*.md`
   - Personal info → update `context/me/` files
4. After processing, moves the original file out of `inbox/` (to its destination or deletes if fully consumed)

**The owner never needs to know the internal folder structure.** Just throw it in inbox.

## Package System — Yar Recipes

Recipes are `.md` files with YAML frontmatter that extend Yar with new capabilities (email integration, HR pipelines, news digests, etc.). They arrive from a trusted source (usually Kobelev / consigliere practice) and are self-contained — each recipe has everything needed for installation.

**Recipe signature:**
```yaml
---
type: yar-recipe
recipe_id: email-calendar
version: 2.0
prerequisites: [node.js, google-account]
---
```

**Flow when a recipe arrives:**
1. Owner drops the file into `inbox/`
2. Owner says "process inbox" (or `/inbox` runs automatically on session start if configured)
3. `/inbox` detects frontmatter `type: yar-recipe` → delegates to `/pkg_installer`
4. `pkg_installer` checks prerequisites, guides through setup, records to `.yar/installed.md`, archives the recipe to `.yar/packages/`

**Where things live after install:**
- Installation registry: `.yar/installed.md` (what, when, version, status)
- Archived recipe files: `.yar/packages/<recipe_id>_v<version>.md`
- Behavior preferences from the recipe: `context/me/preferences.md` (keeps personal settings separate from system state)

**Owner commands:**
- `/pkg_installer <file>` — install a recipe explicitly
- Future: `/pkg_list` — show installed packages, `/pkg_uninstall <id>` — remove a package

---

## Collector Behavior

> **ALWAYS active.** Not a separate agent — a core behavior of the main assistant.

During ANY conversation, watch for and proactively record:

| What to catch | Where to write | When |
|--------------|----------------|------|
| Decisions ("we're going with X") | `context/me/decisions.md` | Immediately, append |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kobelev-ai/yar](https://github.com/kobelev-ai/yar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
