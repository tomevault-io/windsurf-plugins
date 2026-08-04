---
trigger: always_on
description: An [agents.md](https://agents.md) standard file (Linux Foundation / Agentic AI Foundation) - canonical instructions for AI agents working with this repository. Read natively by Cursor, Codex (OpenAI), Jules (Google), Devin / Windsurf (Cognition), Aider, Amp, Factory, GitHub Copilot and other tools on the [official list](https://agents.md/#supported-tools).
---

# AGENTS.md - Patron

An [agents.md](https://agents.md) standard file (Linux Foundation / Agentic AI Foundation) - canonical instructions for AI agents working with this repository. Read natively by Cursor, Codex (OpenAI), Jules (Google), Devin / Windsurf (Cognition), Aider, Amp, Factory, GitHub Copilot and other tools on the [official list](https://agents.md/#supported-tools).

> **For the agent:** if you change anything in this repo, start by reading three files in order: this file (AGENTS.md), [governance/CONSTITUTION.md](./governance/CONSTITUTION.md), [README.md](./README.md). This is not a formality - Patron is a governance product, not ordinary code.

## Project goal

Patron is a **local, GDPR-safe AI agent for law firms**. A zero-cloud, single-user desktop application (Electron): local SQLite by default ([ADR-0053](./governance/adr/0053-sqlite-single-user-zero-cloud.md)) + 6 MCP connectors for Polish and EU law, an audit trail with a hash chain (AI Act art. 12), bring-your-own-model (Gemini / Claude / local Ollama / OpenRouter). Server mode (Postgres + MinIO + Supabase) remains available as an alternative. A fork of [willchen96/mike](https://github.com/willchen96/mike) (AGPL-3.0) - the Patron shell inherits AGPL-3.0 as a derivative work; the MCP connectors are separately licensed under MIT - see [ADR-0002](./governance/adr/0002-dual-license-agpl-shell-mit-connectors.md).

## MateMatic context (HARD CONSTRAINTS)

The repo is maintained by [MateMatic Solutions](https://matematicsolutions.com). Patron is a **regulated product** and is subject to:

- **Attorney and legal-adviser professional secrecy** (PoA art. 6, URP art. 3) - absolute. Patron does not send case files to the cloud without the Operator's consent ([Constitution](./governance/CONSTITUTION.md) Art. 2).
- **GDPR art. 5/25/30/32** - minimization, privacy by design, records of processing activities, security. The data schema (local SQLite in desktop mode - ADR-0053; Postgres `backend/schema.sql` in server mode) is designed for art. 30 and 32.
- **AI Act art. 6 (high-risk AI in law, from 2026-08-02)** + **art. 12 (record-keeping)** - every LLM interaction is logged with a hash chain (ADR-0001).
- **Vendor neutrality** ([Constitution](./governance/CONSTITUTION.md) Art. 4) - Patron does not favor any LLM or provider. Do NOT introduce a dependency on a single provider in the shell code.

## Build and test

```bash
# Backend (Node 20+, TypeScript)
cd backend && npm install && npm run build && npm test

# Frontend (Next.js)
cd frontend && npm install && npm run build && npm test

# Bundle the 6 MCP connectors into the backend image (SERVER mode / docker)
node scripts/bundle-mcp.cjs

# Bundle the 6 MCP connectors + embedder model into the DESKTOP installer (Electron)
# handled in prepare-resources.cjs (stageMcpConnectors + stageEmbedModel),
# requires the 6 built mcp-* repos next to patron/ (MCP_REPOS_DIR, default `..`).
# See ADR-0100. When adding a NODE connector, keep its name in sync in THREE places:
# backend/src/lib/mcp-security/pipeline.ts (APPROVED_PATRON_CONNECTORS),
# desktop/scripts/prepare-resources.cjs (MCP_SERVERS) and mcp-servers.example.json -
# a name mismatch means the typosquat gate + ring-policy block YOUR OWN connector (ADR-0027/0028).
#
# PYTHON connectors (9 EU national ones, Option C - ADR-0136): do NOT freeze per connector,
# but rather ONE bundled standalone CPython + `uv pip install` the 9 into its site-packages
# at build time (stageBundledPython in prepare-resources.cjs). The eli repos live in ~/Projects
# (MCP_PY_REPOS_DIR, not next to patron). 3-way name sync: pipeline.ts APPROVED +
# prepare-resources.cjs MCP_SERVERS_PYTHON + mcp-servers.example.json. Spawn:
# py-runtime/python.exe -s -E -c "from <module>.server import main; main()".
# Build locale: NEXT_PUBLIC_PATRON_LOCALE=en gives the EU-first set + EN tutorial.
cd desktop && npm run build

# Full stack (Docker, requires Supabase + MinIO separately)
cp .env.docker.example .env.docker
# (fill in secrets)
docker compose --env-file .env.docker up -d
```

Tests: 1341/1346 pass (5 todo, 0 fail) as of 2026-07-30 (backend vitest). TSC clean (backend + frontend); frontend `next build` green. **Do not commit if tests fail** - quality gate from the [Constitution](./governance/CONSTITUTION.md) Art. 7.

## Code rules

- **TypeScript strict**. No `any` in new code, no `// @ts-ignore` without a comment explaining why.
- **Audit-first** - every new LLM interaction goes through `backend/src/lib/audit/` (hash chain). A bypass is a critical error.
- **Pseudonymization/anonymization** - sensitive data (PESEL/first name/last name/address) passes through `backend/src/lib/pl-entities/` BEFORE being sent to the LLM. See [ADR-0003](./governance/adr/0003-hey-jude-pseudonim-pipeline.md).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matematicsolutions/patron](https://github.com/matematicsolutions/patron) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-02 -->
