---
trigger: always_on
description: >
---


# iMessage Search

A skill that helps users search their entire iMessage history using natural language.
It exports the macOS Messages database, indexes conversations for efficient scanning,
and lets the user describe what they're looking for in plain English.

**Who this is for:** Anyone on a Mac who wants to dig through their text messages —
they don't need to be technical. The skill walks them through every step.

## How it works (overview for the LLM)

The workflow has three phases:

1. **Setup & Export** — Ensure the user has granted Full Disk Access, then run the
   export script to pull all messages from macOS's `chat.db` into a JSON file.
2. **Index & Chunk** — Run the indexing script to organize messages by conversation
   and create a scannable summary index. This is what makes it possible to search
   through 100K+ messages without blowing up the context window.
3. **Search & Follow-up** — The user describes what they're looking for. Scan the
   conversation index for candidates, load the full threads of matches, and present
   results. Support follow-up queries on the same export without re-exporting.

---

## Phase 1: Setup & Export

### Step 1: Check if an export already exists and whether it needs refreshing

Before doing anything, check whether a previous export exists and how fresh it is:

```bash
ls ~/Downloads/imessage_export/conversations_index.json 2>/dev/null && python3 -c "
import json, os
with open(os.path.expanduser('~/Downloads/imessage_export/conversations_index.json')) as f:
    data = json.load(f)
print('EXPORTED_AT:', data.get('exported_at', 'unknown'))
print('TOTAL_CONVERSATIONS:', data.get('total_conversations', 0))
print('TOTAL_MESSAGES:', data.get('total_messages', 0))
" || echo "NO_EXPORT_FOUND"
```

Then follow this logic:

- **No export found** → Proceed to Step 2 (setup) and Step 3 (export).
- **Export exists but is from a previous day** → Re-export automatically. Tell the
  user: "I found an existing export from [date], but it's not from today. I'll refresh
  it so we have your latest messages." Then run Steps 3–4 (skip Step 2 if Full Disk
  Access is already granted — verify with the access check).
- **Export exists and is from today** → Reuse it. Tell the user: "Using your iMessage
  export from earlier today ([time]). It contains [N] conversations and [M] total
  messages. If you want me to re-export to pick up anything from the last few hours,
  just say 'refresh'."

The goal is that the user's export is always current to at least the start of their
session day. Messages arrive constantly, so a stale export means missed results.

When reusing an existing export, always tell the user when it was last exported so
they can decide if they need a refresh.

### Step 2: Guide the user through Full Disk Access

This is the most important setup step and the one most likely to trip up non-technical
users. Read `references/setup-guide.md` for the detailed walkthrough, then guide the
user through it conversationally.

The short version: the user's terminal app (Terminal, iTerm, Warp, VS Code, etc.)
needs Full Disk Access in System Settings so it can read the Messages database.

Key points to communicate:
- This is a one-time setup — they won't have to do it again
- It's a built-in macOS privacy feature, not a hack
- They'll need to restart their terminal after granting access
- If they're using Claude Code, the terminal running Claude Code is what needs access

After the user confirms they've done this, verify access:

```bash
test -r ~/Library/Messages/chat.db && echo "ACCESS OK" || echo "NO ACCESS"
```

If it fails, walk them through the setup guide again. Common issues:
- They granted access to the wrong app (e.g., Terminal when they're using iTerm)
- They didn't restart the terminal after granting access
- They're on a managed/corporate Mac with restrictions

### Step 3: Run the export

Run the bundled export script. It reads the Messages database and writes a JSON file
to the user's Downloads folder:

```bash
python3 scripts/imessage_export.py export -o ~/Downloads/imessage_export_raw.json
```

Tell the user roughly how long this takes: "This usually takes 10–30 seconds depending
on how many messages you have. It's reading your entire Messages history."

If the script fails, check the error output. The most common issues are:
- Full Disk Access not actually granted (re-check Step 2)
- Python not installed (guide them to install it via `xcode-select --install`)
- Very old macOS version with a different database schema (rare)

### Step 4: Build the conversation index

After the export finishes, run the indexing script to organize messages into
searchable conversations:

```bash
python3 scripts/build_index.py ~/Downloads/imessage_export_raw.json ~/Downloads/imessage_export/
```

This produces:
- `conversations_index.json` — A summary of every conversation (contact, message
  count, date range, last message date, preview of recent messages). This is what
  you'll scan first when searching.
- `conversations/` — A folder of individual conversation files, one per contact/group
  chat. Each file contains the full message thread. You only load these when a
  conversation matches the user's search criteria.

The index is deliberately compact so it fits in a single context window for most users.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [josephofiowa/imessage-search-skill](https://github.com/josephofiowa/imessage-search-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
