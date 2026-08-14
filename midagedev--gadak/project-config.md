---
trigger: always_on
description: Two audiences. Pick your half:
---

# AGENTS.md

Two audiences. Pick your half:

- **[Using the mirror](#using-the-mirror)** — you want answers about issues, or
  you want to comment, transition, or assign one. Most agents stop here.
- **[Developing gadak](#developing-gadak)** — you are changing this repository.

## Using the mirror

gadak keeps a local SQLite mirror of Jira at `~/.gadak/gadak.db` (`--profile x` puts
it under `~/.gadak/profiles/x/`). Reads never touch the network. Writes go to Jira
and re-read the issue into the mirror afterwards.

Four layers. Use the lowest one that answers the question:

| Layer | Use it for | Needs |
| --- | --- | --- |
| **SQL** | anything relational, aggregated, or historical | the file, or `gadak sql` |
| **CLI** | one issue, one search, one write | the `gadak` binary |
| **REST** | the same data from something that is not a shell | `gadak serve` running |
| **MCP** | shell-less clients only (Claude Desktop, etc.) | `gadak mcp` — see [docs/MCP.md](docs/MCP.md) |

### Check freshness before you answer

`issue`, `search`, `comment`, `transition`, `assign`, and `fields` print one line
to **stderr** when the last sync failed or is over an hour old; stdout stays
clean and pipeable. (`sql`, `status`, `open`, `sync`, and others do not.) To
check explicitly:

```bash
gadak status --json
# {"profile":"…","issues":519,"comments":614,"watermark":"…",
#  "version":6,"schema_version":6,"sync_count":1,"first_sync_at":"…"}
```

A `last_error` field means the last sync failed. A quiet project's `watermark`
stalls on its own, so treat an old watermark as "possibly behind", not "broken".
`first_sync_at` / `sync_count` are retention instrumentation (successful syncs
only).

### The one mistake that silently returns nothing

Filter on ids and categories, never on display names. Jira translates
`status.name` and `issuetype.name` per account and ignores `Accept-Language`.

```sql
WHERE status = 'In Progress'          -- WRONG: empty on a Korean account
WHERE status_category = 'inprogress'  -- RIGHT: stable on every site
```

`status_category` is one of `new`, `inprogress`, `done`. `status_id` and
`issue_type_id` are stable too.

### SQL cookbook

The schema in one paragraph: `items` is the source-neutral spine (title,
`body_text`, timestamps); `issues` is the Jira projection, joined on
`issues.item_id = items.id`; **`issues_full` is the agent convenience view**
(`summary` + every `issues` column — prefer it when you need a title);
`comments`, `attachments`, `changelog`, and `links` hang off `items.id`;
`items_fts` is the FTS5 index over titles, bodies, and comment text;
`sync_state` holds freshness. `labels`, `components`, and `fix_versions` are
JSON arrays — reach them with `json_each`. Every column is listed in
`specs/000-product/data-model.md`.

```bash
gadak sql "…"          # tab-separated, read-only
gadak sql --json "…"   # one JSON object per row
gadak sql --csv "…"    # header row plus CSV
```

```sql
-- 1. Someone's open work, most urgent first.
-- Prefer issues_full for titles (summary comes from items.title).
SELECT key, status, priority, summary
FROM issues_full
WHERE assignee_email = 'dana@example.com' AND status_category != 'done'
ORDER BY priority_rank, updated_at DESC;

-- 2. What regressed — reopens are the highest-signal quality metric available
SELECT key, summary, reopen_count, reopened_at
FROM issues_full
WHERE reopen_count > 0 ORDER BY reopen_count DESC, reopened_at DESC LIMIT 20;

-- 3. What is stuck, and for how long
SELECT key, status, ROUND(julianday('now') - julianday(status_changed_at), 1) AS days
FROM issues WHERE status_category = 'inprogress' ORDER BY days DESC LIMIT 20;

-- 4. Has anyone hit this before? (descriptions and comments, one index)
SELECT i.key, it.title FROM items_fts f
JOIN items it ON it.rowid = f.rowid
JOIN issues i ON i.item_id = it.id
WHERE items_fts MATCH 'webhook AND retry' LIMIT 20;

-- 5. Who is loaded, per project
SELECT project_key, COALESCE(assignee, '(unassigned)') AS who, COUNT(*) AS n
FROM issues WHERE status_category != 'done'
GROUP BY project_key, who ORDER BY project_key, n DESC;

-- 6. What is in a release (JSON array column)
SELECT i.key, i.status, i.summary
FROM issues_full i, json_each(i.fix_versions) v
WHERE v.value = '2026.8.0' ORDER BY i.resolved_at;

-- 7. What moved this week, and who moved it
SELECT c.at, c.author, c.field, c.from_value, c.to_value, i.key
FROM changelog c JOIN issues i ON i.item_id = c.item_id
WHERE c.at > datetime('now', '-7 days') ORDER BY c.at DESC LIMIT 50;

-- 8. Untriaged: nobody on it, no priority set
SELECT key, created_at, summary
FROM issues_full
WHERE status_category = 'new' AND assignee_id IS NULL AND priority_rank = 0
ORDER BY created_at LIMIT 30;
```

Rules that come with the file:

- **Never write to the database.** Writes are Jira's job; a row written directly
  is destroyed by the next sync. There is no exception for "just a label".
- **Do not depend on `issues.raw`.** It is an escape hatch shaped by Jira's API,
  not by gadak's contract.
- **Do not poll in a loop.** `sync_state.version` only moves when something
  changed; compare it instead.

### CLI reference

```bash
gadak issue NMB-140                   # fields, description, comments, history, links

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [midagedev/gadak](https://github.com/midagedev/gadak) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
