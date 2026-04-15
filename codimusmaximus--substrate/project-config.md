---
trigger: always_on
description: Manages events, meetings, and scheduling with email invites/reminders via Resend.
---

# Substrate Technical Guide

Personal business operating system framework. Build business systems (CRM, tasks, calendar, notes) with PostgreSQL, Python, Absurd workflows, and AI-native interfaces.

**See [docs/ARCHITECTURE.md](./docs/ARCHITECTURE.md) for full architecture documentation.**

## Database Schema

### notes.notes
Primary table for synced Obsidian vault content.

```sql
CREATE TABLE notes.notes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    file_path TEXT UNIQUE,           -- e.g. "Areas/CRM/Contacts/John.md"
    file_hash TEXT,                  -- SHA256 for change detection
    title TEXT,                      -- From frontmatter or filename
    content TEXT,                    -- Markdown body (no frontmatter)
    frontmatter JSONB DEFAULT '{}',  -- YAML metadata as JSON
    tags TEXT[] DEFAULT '{}',        -- e.g. ['#customer', '#sales']
    created_at TIMESTAMPTZ DEFAULT now(),
    updated_at TIMESTAMPTZ DEFAULT now()
);
```

**Query examples:**
```sql
-- Find by tag
SELECT title, file_path FROM notes.notes WHERE '#customer' = ANY(tags);

-- Search frontmatter
SELECT title FROM notes.notes WHERE frontmatter->>'status' = 'active';

-- Full text search
SELECT title FROM notes.notes WHERE content ILIKE '%product%';

-- Recent notes
SELECT title, updated_at FROM notes.notes ORDER BY updated_at DESC LIMIT 10;
```

### Frontmatter conventions
```yaml
---
status: draft | active | stable | archived
type: note | task | contact | meeting | project
area: sales | product | operations
tags: [customer, priority]
---
```

## Structure

```
.
├── substrate/              # Main codebase
│   ├── core/               # DB, worker, config (infrastructure)
│   ├── integrations/       # Obsidian, HubSpot, etc. (external data)
│   ├── domains/            # CRM, sales, notes (business logic)
│   └── ui/                 # API, chat, web, CLI (interfaces)
├── config/
│   └── sql/                # External SQL schemas
│       └── absurd.sql      # Absurd workflow engine schema
├── libs/
│   └── absurd-sdk/         # Vendored Absurd Python SDK
├── habitat/                # Task monitoring dashboard
├── .secrets/               # Deploy keys, credentials (gitignored)
├── vault/                  # Obsidian vault (symlink)
├── docker-compose.yml
└── pyproject.toml          # Dependencies (absurd-sdk from libs/)
```

## Commands

```bash
# Start all services
docker compose up -d

# Run migrations (after postgres is up)
python -m substrate.core.db.migrate

# Initialize absurd queue
PGPASSWORD=postgres psql -h localhost -U postgres -d substrate -c "SELECT absurd.create_queue('default')"

# Start worker locally (dev)
python -m substrate.core.worker.main

# Start API locally (dev)
uvicorn substrate.ui.api.main:app --reload

# View task dashboard
open http://localhost:7890
```

## Adding a Domain

```bash
mkdir -p substrate/domains/{name}/sql
```

Create:
- `sql/001_init.sql` - schema (`{name}.*` tables)
- `logic.py` - business rules
- `tasks.py` - worker tasks (optional)
- `tools.py` - AI tools (optional)

Then: `python -m substrate.core.db.migrate`

## Adding an Integration

```bash
mkdir substrate/integrations/{name}
```

Create:
- `sync.py` - fetch/push logic
- `tasks.py` - background sync tasks

## Conventions

| Item | Format | Example |
|------|--------|---------|
| Schema | `{domain}.*` | `crm.contacts` |
| Task | `{domain}.{action}` | `crm.enrich` |
| Migration | `NNN_*.sql` | `001_init.sql` |

## Database

```bash
PGPASSWORD=postgres psql -h localhost -U postgres -d substrate
```

## CRUD Web UI

**URL:** http://localhost:8000

A dark-themed web UI for browsing and editing all domain tables.

### Features
- Browse schemas (domains) and tables in sidebar
- View rows with pagination
- View full row details (expands JSON fields)
- Create new rows via JSON editor
- Edit existing rows
- Delete rows

### API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/domains` | List all domains |
| GET | `/api/{schema}/tables` | List tables in schema |
| GET | `/api/{schema}/{table}` | List rows (supports `?limit=N&offset=M`) |
| GET | `/api/{schema}/{table}/{id}` | Get single row |
| POST | `/api/{schema}/{table}` | Create row (JSON body) |
| PATCH | `/api/{schema}/{table}/{id}` | Update row (JSON body) |
| DELETE | `/api/{schema}/{table}/{id}` | Delete row |

### Usage Pattern

```python
# Query via API
import requests

# List notes
res = requests.get("http://localhost:8000/api/notes/notes?limit=10")
notes = res.json()["rows"]

# Create note
res = requests.post("http://localhost:8000/api/notes/notes", json={
    "title": "New Note",
    "content": "# Hello\n\nContent here",
    "tags": ["#work"]
})

# Update note
res = requests.patch(f"http://localhost:8000/api/notes/notes/{note_id}", json={
    "content": "Updated content"
})
```

## Two-Way Sync

The system supports bidirectional sync between PostgreSQL and the Obsidian vault.

### Flow: Vault → DB (read)
```
GitHub repo → git pull → vault/ → parse markdown → INSERT/UPDATE notes.notes
```

### Flow: DB → Vault (write)
```
create_note()/update_note() → INSERT/UPDATE DB → write .md file → git commit → git push
```

### AI Tools (substrate/domains/notes/tools.py)

| Tool | Description |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/codimusmaximus) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
