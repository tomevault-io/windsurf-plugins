---
trigger: always_on
description: This project is a memory database and REST API designed to be driven by you (Claude) via HTTP calls.
---

# BrainDB — Claude Instructions

This project is a memory database and REST API designed to be driven by you (Claude) via HTTP calls.
The API runs at **http://localhost:8000**.

---

## At the Start of Every Session

Before doing any work, consult your memory:

```bash
# 1. Get always-on rules (behavioral guidelines)
curl -s http://localhost:8000/api/v1/memory/rules

# 2. Get context — use multi-query for better coverage
curl -s -X POST http://localhost:8000/api/v1/memory/context \
  -H "Content-Type: application/json" \
  -d '{"queries": ["user profile background expertise", "<what you are working on>"], "max_depth": 3, "max_results": 15}'
```

The context response gives you `items` (ranked memories) and `always_on_rules` (always injected).
Trust higher `final_rank` items more. Check `depth` — depth 0 is a direct match, 1-3 are graph-connected.

Multi-query runs each query independently, merges seeds (keeping the best score per entity), then does one graph expansion on the combined set. Use it to cover multiple angles in a single call.

If results seem weak, retry with reformulated queries (up to 2 times).

---

## Project Structure

```
braindb/
├── braindb/
│   ├── main.py                        # FastAPI app
│   ├── config.py                      # Settings (decay rates, graph params, agent config)
│   ├── db.py                          # get_conn() — one psycopg2 connection per request
│   ├── ingest_watcher.py              # Always-on sidecar: polls data/sources/, triggers agent
│   ├── routers/                       # entities.py, relations.py, memory.py, agent.py
│   ├── schemas/                       # Pydantic models (entities, relations, search)
│   ├── services/                      # search.py, graph.py, context.py, embedding_service.py,
│   │                                  # keyword_service.py, activity_log.py
│   └── agent/                         # Internal agent (LiteLLM + pluggable provider via LLM_PROFILE)
│       ├── agent.py                   # builder + runner (singleton)
│       ├── tools.py                   # 21 @function_tool wrappers
│       └── prompts/system_prompt.md   # baked-in skill content (agent-voiced)
├── alembic/                           # DB migrations (raw SQL, no ORM) — current: 004
├── data/sources/                      # Drop files here — watcher picks them up
│   ├── ingested/                      # Auto-moved here on success
│   └── failed/                        # Auto-moved here on error (with .error.txt sidecar)
├── skills/                            # Shipped Claude Code skills
│   ├── braindb/SKILL.md               # Direct curl-based skill
│   └── braindb-agent/SKILL.md         # Thin wrapper around the agent endpoint
├── docker-compose.yml                 # api + watcher services (external PostgreSQL)
├── .env                               # Real credentials — DO NOT COMMIT
└── BRAINDB_GUIDE.md                   # Full API reference with curl examples
```

**No ORM** — all DB access is raw SQL via psycopg2 `RealDictCursor`.
**No async on the data layer** — plain sync `def` endpoints throughout, except the agent endpoint which is async to support the agent's `Runner.run()` loop.

---

## Saving What You Learn

```bash
# Save a fact
curl -X POST http://localhost:8000/api/v1/entities/facts \
  -H "Content-Type: application/json" \
  -d '{"content": "...", "certainty": 0.9, "source": "user-stated", "keywords": ["k1","k2"], "importance": 0.7}'

# Save a thought / inference
curl -X POST http://localhost:8000/api/v1/entities/thoughts \
  -H "Content-Type: application/json" \
  -d '{"content": "...", "certainty": 0.6, "source": "agent-inference", "context": "what triggered this"}'

# Save a datasource (local file / document)
curl -X POST http://localhost:8000/api/v1/entities/datasources \
  -H "Content-Type: application/json" \
  -d '{"content": "description", "source": "document", "file_path": "/path/to/file", "keywords": ["topic"], "importance": 0.6}'

# Connect two entities
curl -X POST http://localhost:8000/api/v1/relations \
  -H "Content-Type: application/json" \
  -d '{"from_entity_id": "<id>", "to_entity_id": "<id>", "relation_type": "supports", "relevance_score": 0.8, "description": "why"}'
```

Relation types: `supports`, `contradicts`, `elaborates`, `refers_to`, `derived_from`, `similar_to`, `is_example_of`, `challenges`

---

## Useful Endpoints

```bash
# List entities by type / keyword / source
curl -s "http://localhost:8000/api/v1/entities?entity_type=fact&limit=50"
curl -s "http://localhost:8000/api/v1/entities?source=user-stated&limit=50"

# View entity relations
curl -s http://localhost:8000/api/v1/entities/<UUID>/relations

# Explore entity graph tree
curl -s http://localhost:8000/api/v1/memory/tree/<UUID>?max_depth=2

# Activity log (when and how things happened)
curl -s "http://localhost:8000/api/v1/memory/log?limit=20"
curl -s "http://localhost:8000/api/v1/memory/log?operation=create&entity_id=<UUID>"

# Read-only SQL (power queries)
curl -s -X POST http://localhost:8000/api/v1/memory/sql \
  -H "Content-Type: application/json" \
  -d '{"query": "SELECT entity_type, COUNT(*) FROM entities GROUP BY entity_type"}'

# Ingest a file from data/sources/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dimknaf/braindb](https://github.com/dimknaf/braindb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
