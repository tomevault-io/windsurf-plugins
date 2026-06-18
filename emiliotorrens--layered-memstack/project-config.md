---
trigger: always_on
description: >
---


# layered-memstack

3-layer memory system: L1 core → L2 topics/dailies → L3 deep references.

## Architecture

```
MEMORY.md              ← L1: always loaded, ~50-60 lines max (breadcrumbs + pointers)
memory/
├── {topic}.md         ← L2: topic breadcrumbs (viajes, salud, tecnico...)
├── YYYY-MM-DD.md      ← L2: daily notes (auto-generated at 3 AM)
├── archive/           ← dailies older than 14 days
reference/
├── entities.md        ← knowledge graph (people, places, projects, relations)
├── *.md               ← L3: deep dives (loaded on demand via memory_search)
scripts/
└── memory-dedup.js    ← dedup engine
BOOTSTRAP.md           ← compiled snapshot (generated nightly, single read at session start)
```

## Setup

### 1. Create directory structure

```bash
mkdir -p memory/archive reference scripts
```

### 2. Copy dedup script

Copy `scripts/memory-dedup.js` from this skill to the workspace `scripts/` directory.

### 3. Configure memorySearch

Add to `openclaw.json` under `agents.defaults.memorySearch`:

```json5
{
  "extraPaths": ["MEMORY.md", "USER.md", "IDENTITY.md", "memory/", "reference/", "projects/"],
  "sources": ["memory", "sessions"],
  "query": {
    "hybrid": {
      "mmr": { "enabled": true, "lambda": 0.7 },
      "temporalDecay": { "enabled": true, "halfLifeDays": 30 }
    }
  }
}
```

### 4. Create starter files

- **MEMORY.md** — see `references/memory-template.md`
- **reference/entities.md** — see `references/entities-template.md`
- **INDEX.md** — catalog of all files with tags and line counts

### 5. Enable Dreaming (OpenClaw 2026.4.8+)

Dreaming replaces the manual 3 AM auto-summary cron. Enable it in OpenClaw config:

```bash
openclaw config patch '{"plugins":{"entries":{"memory-core":{"config":{"dreaming":{"enabled":true,"frequency":"0 3 * * *","timezone":"Europe/Madrid"}}}}}}'
```

Then restart the gateway. OpenClaw will automatically create and manage the nightly consolidation sweep.

### 6. Set up remaining crons

Run the setup script to create the weekly audit (and optional MCP audit) cron:

```bash
bash scripts/setup-crons.sh --tz Europe/Madrid --channel telegram --to "CHAT_ID"
```

Options:
- `--tz IANA` — timezone (default: Europe/Madrid)
- `--channel telegram|whatsapp|discord` — delivery channel for summaries
- `--to CHAT_ID` — delivery target (Telegram chat ID, phone number, etc.)
- `--model alias` — model override (default: uses your configured default)
- `--dry-run` — show what would be created without creating
- `--mcp-audit` — also create the nightly MCP memory security audit cron

Or create them manually (see Cron Setup below).

## Layer Rules

| Layer | When to load | What goes here | Size target |
|-------|-------------|----------------|-------------|
| L1 | Every session start | Breadcrumbs + pointers to L2/L3. Core facts, active project names, pending items. **No detail here.** | ~50-60 lines |
| L2 | Today + yesterday auto-loaded; older via search | Topic summaries, daily notes with decisions/actions/facts | No limit |
| L3 | Only via memory_search | Deep dives, travel details, health data, technical docs | No limit |

### L1 Writing Rules

- MEMORY.md is **breadcrumbs + pointers only**. Detailed info goes in reference/ or memory/ files.
- If something already has a pointer in L1, update the reference file — NOT MEMORY.md.
- Before writing to MEMORY.md, always check for duplicates first:
  ```bash
  node scripts/memory-dedup.js --query "text to check"
  # Exit 0 = duplicate (skip), Exit 1 = new (safe to add)
  ```
- After any write, run dedup fix:
  ```bash
  node scripts/memory-dedup.js --fix
  ```
- Use TTL comments for time-bound items: `<!-- ttl:YYYY-MM-DD -->`
- Keep entries atomic — one fact per line
- Use L2 breadcrumbs to point to topic files: `Viajes → memory/viajes.md`

### L2 Writing Rules

- Daily notes: `## Checkpoint [HH:MM]` sections with decisions, actions, atomic facts
- Topic files: organized by subject, mid-level detail
- Archive dailies older than 14 days to `memory/archive/`

### L3 Writing Rules

- Detailed docs only — don't duplicate L1/L2 content
- Update `reference/entities.md` when new entities appear
- Structure: sections by entity type (people, places, projects, etc.)

## Dedup Engine

`scripts/memory-dedup.js` — token-based similarity without embeddings.

### Modes

```bash
# Check for duplicates (read-only)
node scripts/memory-dedup.js --check

# Fix: remove exact dupes, mark semantic dupes with <!-- dup? -->
node scripts/memory-dedup.js --fix

# Query single text against MEMORY.md
node scripts/memory-dedup.js --query "GitHub configured"
# Exit 0 = duplicate, Exit 1 = new

# Batch query: filter lines from file
node scripts/memory-dedup.js --query-batch /tmp/candidates.txt
```

### Options

- `--file path` — target file (default: MEMORY.md)
- `--threshold 0.65` — similarity threshold (default: 0.65)
- `--verbose` — show comparison details

### Algorithm

Jaccard similarity + containment ratio + entity overlap + segment-level comparison.
Entities extracted: dates, version numbers, hex IDs, phone numbers, amounts, chat IDs.
Threshold 0.65 balances false positives vs missed duplicates.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [emiliotorrens/layered-memstack](https://github.com/emiliotorrens/layered-memstack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
