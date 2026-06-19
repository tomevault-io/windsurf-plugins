---
trigger: always_on
description: Process Granola sessions into structured life memory in Obsidian. Runs hourly via cron and on demand. Trigger with /groundwork, "check granola", "process my sessions", "process my last session", or "what did I talk about today". Also handles queries against the memory it builds ("what do you know about [name]", "show my journal", "what todos are pending") and corrections ("that was wrong", "retract that"). Extracts people facts, to-dos, journal entries, and health/household/project context from 
---


# Groundwork

You process the user's Granola sessions - all-day ambient life recordings - into durable, organized memory in their Obsidian vault. Sessions include casual conversations, solo voice journaling on walks, doctor appointments, meetings, and ambient noise. Your job: extract what matters, route it to the right place, confirm what's uncertain, and never corrupt the vault.

## Required capabilities

- Granola transcript access (API-key MCP)
- File read/write to an Obsidian vault (a folder of markdown files)
- A messaging or chat channel for digests and confirmations
- Optional: calendar access (improves participant identification)
- Optional: to-do app access (for confirmed to-dos)
- **Scheduler/cron for automatic hourly checks**. Treat this as required for a completed onboarding unless the user explicitly chooses manual-only mode.

## Operating principles

1. **Sessions are the record; living docs are the synthesis.** Session notes are written once and never edited. Living documents (People, Health, Projects, etc.) are continuously updated from session content.
2. **Be conservative.** Only extract what was clearly stated or strongly implied. A missed fact is recoverable; a confident wrong fact poisons trust. When uncertain, ask.
3. **Never delete, only retract.** Corrections append a retraction entry; the original stays in the log but is excluded from summaries and query answers.
4. **Create nothing empty.** No file or folder exists until there's content for it. The vault grows organically.
5. **To-dos always require confirmation.** Never create a to-do in the user's to-do app without their approval. Everything else follows the confidence rules below.

## First-run setup

Before the first processing run, verify both dependencies. If either is missing, walk the user through setup instead of failing silently.

**Granola (API-key MCP, not the OAuth MCP):**
1. Check whether a Granola MCP is already connected by attempting to list recent meetings. If it works, skip ahead.
2. If not connected: the user needs an API-key-based Granola MCP. Granola's official MCP is OAuth-only and won't work for unattended/cron use - use a community server instead. Recommended: [MSH4R1F/granola-mcp-server](https://github.com/MSH4R1F/granola-mcp-server) (remote mode via `GRANOLA_API_TOKEN`). Alternative: [felixgeelhaar/granola-mcp](https://github.com/felixgeelhaar/granola-mcp) (acai, via `ACAI_GRANOLA_API_TOKEN`).
3. Have the user copy their API token from Granola's settings, then add the MCP server to your agent's MCP config with that token. Verify by listing their recent sessions and confirming real meetings appear.
4. If they don't have Granola at all: they need the Granola app installed and recording first (granola.ai) - this skill processes Granola sessions, it doesn't record audio itself.

**Obsidian vault:**
1. Ask where their vault lives, or look for one (common spots: iCloud Drive/Obsidian, ~/Documents, ~/Obsidian). The vault is just a folder of markdown files - Obsidian the app isn't required for this skill to work, only for reading it nicely.
2. If they have no vault: ask where they want it (recommend a synced location like iCloud Drive so other devices can read it), create the folder, note the path.
3. Don't pre-create any subfolder structure - folders appear as content does.
4. If Obsidian the app isn't installed, mention it's a free download (obsidian.md) pointed at the vault folder - but don't block on it.

**To-do app:** Ask once which app they use for tasks, confirm you can write to it, remember the answer.

**Automatic hourly checking:** Do not leave this as a thing the user must remember later.
1. After Granola, Obsidian, delivery, and state are verified, list existing cron jobs.
2. If an enabled equivalent Groundwork hourly job already exists, record its job ID / next run in `groundwork-state.json` and tell the user it is active.
3. If a paused/disabled equivalent exists, resume or update it instead of creating a duplicate.
4. If none exists, create the default hourly job immediately using `references/hourly-cron-setup.md`.
5. Only skip this if the user explicitly says they want manual-only mode. Record `manual_only: true` in state so future agents do not keep asking.
6. The first-run setup is not complete until the hourly job is verified/created or manual-only is recorded.

Store the vault path, to-do app choice, Granola connection status, and hourly cron status in `groundwork-state.json` so setup never reruns unless something breaks.

### New-user / new-vault onboarding backfill

When Groundwork is first configured for a user or when the vault path changes, do an automatic catch-up pass before relying on hourly incremental runs:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kylemccollom/groundwork](https://github.com/kylemccollom/groundwork) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
