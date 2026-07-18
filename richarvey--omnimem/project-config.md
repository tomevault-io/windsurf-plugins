---
trigger: always_on
description: Self-hosted semantic memory MCP server for Claude Code. Provides persistent memory across sessions via four namespaces: episodic (decisions, bugs, patterns), project context (stack, goals, state), knowledge base (RSS articles auto-summarised by Claude Haiku), and preferences (prescriptive rules extracted from conversation, e.g. "always update README after a feature"). v6 adds a fifth, derived namespace: compiled skills (`mem:skill:`) — SKILL.md documents distilled from experience and graveyard m
---

# OmniMem Development Guide

## What is this?

Self-hosted semantic memory MCP server for Claude Code. Provides persistent memory across sessions via four namespaces: episodic (decisions, bugs, patterns), project context (stack, goals, state), knowledge base (RSS articles auto-summarised by Claude Haiku), and preferences (prescriptive rules extracted from conversation, e.g. "always update README after a feature"). v6 adds a fifth, derived namespace: compiled skills (`mem:skill:`) — SKILL.md documents distilled from experience and graveyard memories per domain, gated behind a propose-and-accept write path.

**Version**: 6.3.2
**Stack**: Python 3.12, FastMCP (SSE transport), Valkey + valkey-search (HNSW vectors), sentence-transformers (all-MiniLM-L6-v2, 384-dim), Anthropic API (Claude Haiku for RSS summarisation), Pydantic v2, Docker Compose, APScheduler, feedparser, PyTorch CPU-only

## Project Structure

```
mcp_server/           # MCP server — FastMCP SSE transport
  server.py           # Entry point: init store/embedder/lifecycle/pipeline, register tools
  memory/             # Core engine (shared with web_ui)
    store.py          # ValkeyStore: connection pool, HNSW vector indexes, CRUD
    embedder.py       # Singleton SentenceTransformer (all-MiniLM-L6-v2, 384-dim)
    lifecycle.py      # MemoryState enum, state transitions, topic suppression
    recall.py         # RecallPipeline: abandoned fast-path → vector search → scoring
    dedup.py          # Cosine similarity duplicate detection (threshold 0.92)
    maintenance.py    # Auto-maintenance: dedup + contradiction scan on briefing interval
    contradiction.py  # Tier 1 heuristic + optional Tier 2 Claude Haiku API
    skills.py         # v6 skill compiler engine: domain pools, lesson clustering, SKILL.md rendering, diffs
    skill_compiler.py # Shared propose-and-accept compile flow (used by MCP compile_skill AND the web UI)
  tools/              # 30+ MCP tool implementations
    core.py           # remember, recall, recall_index, recall_detail, deprioritise, archive, forget
    project.py        # set/get/update/compile project_context, list_projects, delete_project
    experience.py     # record_experience, log_abandoned, warn_if_abandoned
    briefing.py       # Session-start 5-in-1 aggregation
    audit.py          # memory_audit, explain_memory, why_did_you_mention
    backup.py         # dump_to_file, restore_from_file, list_backups
    contradiction.py  # check_contradictions tool
    topics.py         # suppress/unsuppress/list_suppressions
    skills.py         # compile_skill (propose/write gate), find_skills, get_skill, bless + briefing surfaces
  tests/              # pytest with in-memory fakes (no Docker needed)
    conftest.py       # FakeValkeyClient, FakeEmbedder, FakeStore fixtures + web_client
                      # (TestClient over the real web_ui app; neutralises load_dotenv so
                      # a production .env further up the tree can't leak into tests)

web_ui/               # Starlette + Jinja2 + htmx dashboard
  app.py              # ASGI app setup, route mounting
  deps.py             # Shared init (mirrors server.py pattern)
  routes/             # 17 route modules (memories, search, projects, skills, feeds, telemetry, metrics, etc.)
  templates/          # Jinja2 templates with htmx partials
  static/             # htmx.min.js, style.css

rss_worker/           # Background RSS ingestion
  worker.py           # APScheduler entry + feeds.yml file watcher
  ingester.py         # Fetch → strip HTML → summarise → embed → store
  summariser.py       # Claude Haiku summaries or truncation fallback
  feeds.yml           # Feed definitions (url, name, topics, optional project label)

claude_config/        # CLAUDE.md template for end-users to copy into their projects
scripts/              # health_check.sh, restore_backup.sh
```

## Running Locally

```bash
cp .env.example .env   # Edit: set VALKEY_PASSWORD, optionally ANTHROPIC_API_KEY
docker compose up -d
```

- MCP server: `http://localhost:8765/mcp`
- Web UI: `http://localhost:8080`
- Valkey: `localhost:6379`

## Running Tests

```bash
cd mcp_server && pytest tests/
```

Tests use in-memory fakes (FakeValkeyClient, FakeEmbedder) — no running Valkey required.

For Docker-based tests: `docker compose -f docker-compose.test.yml up --build`

## Key Architecture Decisions

- **ULIDs** for memory keys (sortable, collision-free)
- **SSE transport** (stateless, simpler than WebSocket for MCP)
- **Valkey** over Redis (open source fork)
- **CPU-only PyTorch** (no GPU dependency)
- **Shared `memory/` package** between MCP server and web UI (no code duplication)
- **Debian-slim Docker base** — Alpine doesn't work (PyTorch has no musllinux wheels)
- **In-memory fakes** for testing (no Docker-in-tests complexity)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [richarvey/OmniMem](https://github.com/richarvey/OmniMem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
