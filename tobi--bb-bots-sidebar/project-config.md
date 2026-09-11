---
trigger: always_on
description: Work on `main` in the installed Bots Sidebar source checkout. Confirm its path
---

# Independent Bots development

Work on `main` in the installed Bots Sidebar source checkout. Confirm its path
with `bb plugin list` before editing. Do not edit or reload older personal-workspace
worktrees. Build and reload `bots-sidebar` from this checkout; verify the installed
source before deploying. The owner requested private, ID-keyed state,
not bot-home projects or special execution directories. Explicit user-created
work projects are allowed, but never create a backing project for bot storage.
Keep all UX refinements intact.
A code-only downgrade is not a data rollback: preserve the plugin DB backup before
restoring an older metadata format.

Private state design: use the plugin SQLite database as the durable authority,
with private per-bot Markdown/JSON exports under its own data directory. Embed
identity/instructions/memory into agent context; tools infer bot ID from explicit
thread bindings. Ordinary bot chats use the personal project with a normal personal
workspace. No new backing projects, no state files in user repositories. Preserve
existing conversation histories and track legacy home project IDs for compatibility.

## Plugin identity

- Runtime ID: `bots-sidebar`; package: `bb-plugin-bots-sidebar`; display name:
  Bots Sidebar. The sidebar heading can remain Bots.
- `bots` is another marketplace plugin's ID. Do not add a runtime alias or read
  its private directory/KV automatically. Our old local namespace was migrated
  explicitly with backups. Historical thread origin IDs remain unchanged; durable
  bindings identify our old conversations. New starts must use `bb.pluginId`.
- Keep native tool names, SQL table names, RPC names, and existing `bots:` client
  preference/composer-draft keys stable. They are not runtime plugin identities.

## Data and state contract

- A bot has its own ID/name/private state and many linked projects. Never infer identity,
  activity, visibility, or thread ownership from a linked project's ID.
- A work project may have exactly one owner, independently of its many members.
  Persist ownership in `bot_project_owners` with a unique project key. Owning
  implies membership; leaving releases ownership; releasing keeps membership.
  Never automatically promote an existing membership to ownership.
- Existing thread bindings win, then explicit server-issued bot-start tokens,
  then parent/fork context. Only otherwise default NEW project threads to its
  owner. Never reassign old history or capture threads created before a claim.
  Explicit bot starts and new inherited contexts join that bot to the work project
  without changing its owner. Personal/legacy-home projects cannot be owned.
  Unassigned conversations still belong in Chats. Preserve all main pointers.
- Preserve the first-turn dispatch binding before agent configuration; a
  fire-and-forget thread.created event alone is insufficient. Configuration
  lookups are read-only; they must not preempt an inherited binding + join.
- Legacy migration is snapshot-based and read-only for old projects. Keep old
  main pointers/history. Import v2 state from legacy homes once when available,
  preserving a cached fallback and warning if unavailable; never overwrite newer
  private state on a later retry.
- Bot state belongs to plugin storage, never a working repository. State writes
  use SQLite transactions and expected hashes/revisions. Markdown/JSON files are
  private exports, not a second write interface or runtime working directory.
- Editing a bot must not rename or relocate linked projects. Native tools act
  only on the current bot and should not expose storage paths to the agent.
- `bot_update_state` is the durable-write tool: always require target + action.
  Identity/set requires a fresh revision. Memory is at most 3000 characters on every
  write; append/forget operates on exact single-line facts. Memory overwrite requires
  content + expectedSha256 from a fresh read, and never retries whole-document bytes
  against a newer hash. Overflow errors explain read/condense/overwrite. Preserve
  older oversized memories for reading and explicit compaction; settings set/unset only changes selected top-level keys;
  project join/leave/own/release changes membership/ownership atomically, never
  project files or historical conversation bindings. Reapply semantic edits on bounded CAS
  retries, never retry stale whole-file bytes. No shared-memory scope or routines.

# Bots sidebar UX contract

Preserve the approved interaction model below. An older source snapshot once
reintroduced removed controls during development. Do not restore old snapshots
or reintroduce hover toolbars when making unrelated changes.

- Clicking/selecting a bot opens its main and reveals all working/waiting top-level
  conversation trees FIRST, plus at most FIVE inactive trees. Active work does not
  consume those five slots. Put remaining inactive roots behind a separate N Other
  toggle, collapsed by default; do not render a Topics heading. Keep viewed/pinned
  branches within the five inactive slots, never extra. A busy descendant keeps
  its root out of overflow. Each conversation's children stay independently collapsed.
  The bot-row chevron remains ONLY for the main conversation's children.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tobi/bb-bots-sidebar](https://github.com/tobi/bb-bots-sidebar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
