---
trigger: always_on
description: This file is the repository-wide source of truth and current operational map for
---

# Repository Guide for AI Coding Agents

This file is the repository-wide source of truth and current operational map for
Codex, Claude Code, Kimi Code, and other coding agents. `AGENTS.zh-CN.md` is the
synchronized human-facing Chinese translation. `CLAUDE.md` imports this file for
Claude Code.

## 1. Authority and Working Contract

- Read this file and any nearer `AGENTS.md` before acting in a subtree. A nearer
  guide may add subtree-specific commands or constraints but may not weaken root
  rules.
- Source and executable checks describe current behavior. Approved designs
  describe intended behavior. Report drift instead of silently choosing one.
- Reply in Chinese unless the user requests another language. Keep source
  identifiers, API names, and test names in English. Match the surrounding
  language for comments and docstrings; Chinese user-facing text is part of the
  current product behavior.
- Read the owning implementation, contracts, documentation, and relevant checks
  before editing. Keep one change focused on one approved objective.
- Preserve user work. Do not overwrite, revert, delete, stage, or publish
  unrelated changes.
- Before a feature, refactor, deletion, dependency or schema change, batch edit,
  global configuration change, new top-level directory, or other high-impact
  change, present the goal, users or stakeholders, MVP, non-goals, file scope,
  acceptance criteria, and risks or tradeoffs for approval.
- Never expose credentials or private endpoints in source, command arguments,
  fixtures, logs, screenshots, manifests, or results. Confirm scope before an
  external write or irreversible action.

## 2. Repository Profile

This repository is a personal, public reference implementation of a multi-agent
AIOps diagnosis workbench for OnCall and SRE scenarios. It accepts user reports
or Alertmanager events, selects a Skill playbook, gathers evidence through RAG
and MCP tools, and emits traceable Markdown reports.

The current product generation is called **V3**. V3 adds a background task
pipeline, Redis Streams, worker processes, Postgres facts, fast/deep diagnosis
modes, evidence audit records, approval structures, an LLM Wiki, retrieval
evaluation, and concurrency testing. It is a reference implementation, not a
claim of production readiness.

Primary stakeholder: the repository owner and maintainer. Secondary stakeholders
are users who run the demo and contributors who need reproducible project facts.

### Keep this map current

This guide describes how the repository works now; it is not a changelog,
backlog, or copy of every implementation detail. Update the relevant root or
module guide in the same change when code changes the top-level layout, module
ownership, service topology, primary flow or entry point, dependency direction,
public protocol, compatibility or security boundary, canonical command, or
validation requirement.

Do not churn this guide for an internal refactor or small bug fix that leaves
those facts unchanged. Update `README.md` for user-facing setup or behavior. Keep
durable detail that cannot fit this orientation map in
`docs/ARCHITECTURE.md`, and link instead of duplicating it.

### Verified stack

- Python 3.12 is the container baseline (`Dockerfile`). The repository has no
  package metadata declaring a wider supported Python range.
- FastAPI and Uvicorn provide the HTTP/SSE application.
- LangGraph runs the fast and deep diagnosis graphs.
- Milvus stores vectors; Redis stores queue/runtime coordination; Postgres stores
  incident and diagnosis facts.
- MCP services expose system, web search, Windows event log, network, and Docker
  tools.
- `open-webSearch-main/` is a vendored Apache-2.0 third-party project with its own
  Node.js build and documentation.

## 3. Canonical Setup and Commands

Create local configuration before commands that load Compose application
services:

```bash
python3.12 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
```

Configure model credentials and an embedding provider in `.env`. Model names and
API keys must use the same provider. Never commit `.env`.

Infrastructure only:

```bash
docker compose up -d
```

Complete containerized stack (API, three workers, MCP services, and
infrastructure):

```bash
docker compose --profile app up -d --build
```

Local macOS/Linux stack with containerized infrastructure and local Python
processes:

```bash
bash scripts/run_all.sh
bash scripts/stop_all.sh
```

The Windows `run.ps1` launcher is a local compatibility entry point. It does not
start the complete V3 Postgres/worker topology; use the Compose `app` profile for
the full stack.

Knowledge-base ingestion and evaluation:

```bash
python scripts/ingest_kb_corpus.py --dry-run
python scripts/ingest_kb_corpus.py --reset --batch 8
python benchmark/run_benchmark.py retrieval --k 3
python benchmark/run_benchmark.py ragas --limit 5
```

`ragas`, real diagnosis, ingestion with remote embeddings, and some health checks
may call paid or external providers. Do not run them merely as a static quality
gate; confirm credentials, cost, data scope, and service readiness first.

## 4. Information and Directory Ownership

| Path | Primary responsibility |
| --- | --- |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kkkirito-123/Mutil-Rag-Agent](https://github.com/Kkkirito-123/Mutil-Rag-Agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
