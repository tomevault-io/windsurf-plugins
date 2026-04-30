---
trigger: always_on
description: A personal knowledge agent that learns how you work and builds a compounding knowledge base. Navigates heterogeneous data sources using native query interfaces. Raw data goes in, a structured wiki comes out, and every interaction compounds.
---

# Pal

A personal knowledge agent that learns how you work and builds a compounding knowledge base. Navigates heterogeneous data sources using native query interfaces. Raw data goes in, a structured wiki comes out, and every interaction compounds.

The canonical specification is `docs/SPEC.md`. All other documentation derives from it.

## Architecture

- Team definition: `pal/team.py` (Pal team leader, Coordinate mode)
- Member agents: `pal/agents/navigator.py` (Navigator), `pal/agents/researcher.py` (Researcher), `pal/agents/compiler.py` (Compiler), `pal/agents/linter.py` (Linter), `pal/agents/syncer.py` (Syncer, conditional)
- Shared settings: `pal/agents/settings.py` (DB, knowledge bases)
- Config: `pal/config.py` (all env var reads and feature flags)
- Instructions: `pal/instructions.py` (prompt strings + builders)
- Tools: `pal/tools/` (knowledge, ingest, wiki, build)
- API server: `app/main.py` (FastAPI + AgentOS + optional Slack interface)
- Custom router: `app/router.py` (/context/reload, /wiki/compile, /wiki/lint, /wiki/ingest, /sync/pull)
- Database: PostgreSQL + pgvector (knowledge, learnings, user data, sessions)

## Team Structure

```
Pal (Team, Coordinate, gpt-5.4)
├── Navigator  — routes queries, reads wiki, handles email/calendar/SQL/files
├── Researcher — web search, source gathering, writes to raw/
├── Compiler   — reads raw/, writes wiki articles, maintains index
├── Linter     — health checks, finds gaps, suggests research
├── Syncer     — commits and pushes context/ changes to GitHub (conditional)
└── [leader responds directly for greetings/simple questions]
```

- **Pal (leader):** Triages requests, delegates to specialists, posts to Slack. Chains Syncer after file-creating workflows.
- **Navigator:** SQLTools, FileTools, MCPTools (Exa), GmailTools, CalendarTools, update_knowledge, wiki read tools, read_manifest
- **Researcher:** FileTools, ParallelTools (parallel_search, parallel_extract), update_knowledge, ingest tools (ingest_url with auto-fetch, ingest_text, read_manifest) (conditional — requires PARALLEL_API_KEY)
- **Compiler:** FileTools, update_knowledge, ingest tools (read_manifest, update_manifest_compiled), wiki tools (read/update index, read/update state)
- **Linter:** FileTools, MCPTools (Exa), update_knowledge, wiki tools (read index, read/update state)
- **Syncer:** sync_push, sync_pull, sync_status (conditional — requires GITHUB_ACCESS_TOKEN + PAL_REPO_URL)

Navigator has both `pal_knowledge` and `pal_learnings`. Researcher, Compiler, and Linter share `pal_knowledge`. Syncer has no knowledge base.

## Key Concepts

- **Navigation over search:** Each source keeps its native query interface. No flattening into a single vector store.
- **Knowledge base pipeline:** Raw data → `context/raw/` → Compiler → `context/wiki/` → Navigator answers questions
- **Dual knowledge system:** `pal_knowledge` (metadata routing) + `pal_learnings` (operational memory). Both PgVector hybrid search.
- **Wiki index as routing layer:** Navigator reads `wiki/index.md` first for knowledge questions, then pulls specific articles.
- **Manifest tracking:** `.manifest.json` in raw/ tracks ingest/compile state. Compiler only processes `compiled: false` entries.
- **Execution loop:** Classify → Recall → Read → Act → Learn
- **Thread-as-session:** Slack thread timestamps = session IDs.
- **Startup schedule registration:** All scheduled tasks register in `app/main.py` lifespan (idempotent, `if_exists="update"`).
- **Git-backed persistence:** Context/ is synced to GitHub via the Syncer agent. No volumes needed — git is the persistence layer. Push is event-driven (after work), pull is scheduled (every 30 min). Requires `GITHUB_ACCESS_TOKEN` + `PAL_REPO_URL`.
- **Production authorization:** `authorization=runtime_env == "prd"` enables RBAC in production. Requires `JWT_VERIFICATION_KEY` from os.agno.com.
- **Governance:** No email sending (draft only). No file deletion. No cross-user data access. External calendar events require confirmation.

## Structure

```
pal/
├── app/
│   ├── main.py              # AgentOS + Slack interface + scheduler + lifespan schedule registration
│   ├── router.py            # /context/reload, /wiki/compile, /wiki/lint, /wiki/ingest, /sync/pull
│   └── config.yaml          # Quick prompts for web UI
├── pal/
│   ├── team.py              # Pal team definition (leader)
│   ├── config.py            # Environment variables and feature flags
│   ├── instructions.py      # Instruction strings + builders
│   ├── paths.py             # Shared path constants
│   ├── agents/
│   │   ├── navigator.py     # Navigator agent (core ops + wiki Q&A)
│   │   ├── researcher.py    # Researcher agent (web → raw/)
│   │   ├── compiler.py      # Compiler agent (raw/ → wiki/)
│   │   ├── linter.py        # Linter agent (wiki health checks)
│   │   ├── syncer.py        # Syncer agent (git commit + push)
│   │   └── settings.py      # Shared DB, knowledge bases
│   └── tools/
│       ├── knowledge.py     # update_knowledge tool
│       ├── ingest.py        # ingest_url, ingest_text, manifest tools

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agno-agi/pal](https://github.com/agno-agi/pal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
