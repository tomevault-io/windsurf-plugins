---
trigger: always_on
description: > Read this file before every session. It has everything Claude needs to build the Trackly backend.
---

# Trackly — Claude Backend Context
> Read this file before every session. It has everything Claude needs to build the Trackly backend.
> Work through weeks in order. Complete Week 1 before starting Week 2.

---

## Project Overview

Trackly is an AI-powered Knowledge Management System replacing Jira + Confluence for 3SC Solutions.
46 engineers · 8 PODs · Local hosting · Built-in AI called NOVA (no external APIs).

**Tagline:** The work OS for modern teams.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Backend | FastAPI (Python 3.12) + Uvicorn |
| Database | PostgreSQL 15 + pgvector extension |
| ORM | SQLAlchemy 2.0 |
| Migrations | Alembic |
| AI Engine | NOVA — Llama 3.1 8B via Ollama (`http://ollama:11434`) |
| Embeddings | sentence-transformers `all-MiniLM-L6-v2` (384-dim, local) |
| Reranker | cross-encoder/ms-marco-MiniLM-L-6-v2 (local) |
| Auth | JWT HS256 24h + bcrypt 12 rounds |
| Scheduler | APScheduler |
| Hosting | Docker Compose (fully local) |

---

## Coding Conventions — Follow Always

- **Async routes** — all FastAPI routes are `async def`
- **No raw SQL strings** — use SQLAlchemy ORM or `text()` with bound params
- **Auth on every route** — `Depends(get_current_user)` — no exceptions
- **Org scoping** — every DB query filters by `org_id` — no cross-tenant leakage
- **Pydantic v2** — request body = `...Request`, response = `...Response`
- **Error handling** — raise `HTTPException(status_code=..., detail="...")`
- **Soft deletes** — use `is_deleted = True`, never hard DELETE on tickets/comments
- **Parameterised queries** — never interpolate user input into SQL strings

---

## Environment Variables

```env
DATABASE_URL=postgresql://trackly:trackly@postgres:5432/trackly
JWT_SECRET=<openssl rand -hex 32>
EMBEDDING_MODEL=all-MiniLM-L6-v2
NOVA_MODEL=llama3.1:8b
NOVA_BASE_URL=http://ollama:11434
NOVA_temperature=0
NOVA_MAX_TOKENS=1500
SMTP_HOST=mailhog
SMTP_PORT=1025
```

---

## Already Built — Do Not Rebuild

| File | What it does |
|---|---|
| `main.py` | FastAPI app setup, route registration, CORS |
| `auth.py` | JWT login, `get_current_user`, `get_admin` dependencies |
| `database.py` | SQLAlchemy engine, session, pgvector setup |
| `models.py` | Pydantic models for existing endpoints |

### Existing DB Tables
- `organisations` — org config (`id`, `name`, `jira_url`, `jira_token`)
- `users` — auth + hierarchy (`id`, `name`, `email`, `role`, `pod`, `emp_no`, `reporting_to`, `password_hash`, `org_id`)
- `jira_tickets` — synced tickets (`id`, `org_id`, `jira_key`, `summary`, `assignee`, `pod`, `client`, `hours_spent`, `status`, `issue_type`, `priority`)
- `worklogs` — time entries (`id`, `ticket_id`, `author`, `author_email`, `hours`, `log_date`)
- `manual_entries` — manual time logs (`id`, `user_id`, `org_id`, `activity`, `hours`, `entry_date`, `pod`, `client`, `status`)
- `ticket_embeddings` — pgvector (`id`, `ticket_id`, `embedding vector(384)`, `content_snippet`, `updated_at`)

### Existing API Endpoints
```
POST /api/auth/login
GET  /api/auth/me
POST /api/auth/set-password
GET  /api/summary
GET  /api/tickets
GET  /api/activity
GET  /api/filters
GET  /api/team
GET  /api/engineer-stats
POST /api/manual-entries
GET  /api/export/monthly
GET  /api/export/fy
GET  /api/users            (admin only)
POST /api/employees/sync
```

---

## Role Permissions

| Role | Tickets | Wiki | Reports | Admin |
|---|---|---|---|---|
| admin | Full CRUD | Full CRUD | All | Full |
| engineering_manager | Full CRUD own PODs | Full CRUD | Own PODs | Read |
| tech_lead | Full CRUD own PODs | Create/Edit | Own POD | None |
| team_member | Create + own | Create/Edit | Own only | None |
| finance_viewer | Read only | Read only | All read | None |

---

---

# ✅ WEEK 1 — NOVA Core + Ticket CRUD + AI Analysis
> **Deadline: April 17** | Complete these in order — each step unblocks the next.

## What to build this week

```
backend/
├── ai/
│   ├── nova.py                ← START HERE (everything depends on this)
│   ├── ticket_intelligence.py ← Step 2
│   └── search.py              ← Step 3
├── routes/
│   └── tickets.py             ← Step 4
└── alembic/versions/
    └── xxx_add_ticket_tables.py  ← Step 5 (comments + attachments)
```

---

## Step 1 — `backend/ai/nova.py` (NOVA Core Engine)

Build this first. Every AI feature in the project calls this file.

```python
"""
NOVA — Neural Orchestration & Velocity Assistant
Trackly's built-in AI. 100% local. Zero external API.
"""
import httpx
import json
from sentence_transformers import SentenceTransformer, CrossEncoder
from typing import Optional
import logging

logger = logging.getLogger(__name__)

EMBEDDING_MODEL = SentenceTransformer("all-MiniLM-L6-v2")
RERANKER_MODEL  = CrossEncoder("cross-encoder/ms-marco-MiniLM-L-6-v2")
OLLAMA_BASE_URL = "http://ollama:11434"
OLLAMA_MODEL    = "llama3.1:8b"

NOVA_SYSTEM_PROMPT = """You are NOVA, the built-in AI assistant for Trackly —
a work management platform used by engineering and cross-functional teams at 3SC Solutions.
Be concise, accurate, and helpful. When analysing tickets be specific.
When generating documents use markdown formatting.
Always ground your answers in the provided context when available."""


async def chat(
    user_message: str,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anandverma0908/timesheet-tracker-backend](https://github.com/anandverma0908/timesheet-tracker-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
