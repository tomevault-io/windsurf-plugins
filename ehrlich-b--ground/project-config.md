---
trigger: always_on
description: Ground is a source-anchored encyclopedia of weighted facts. Every claim traces back to verbatim quotes from cited sources. Sources carry credibility scores, exposed as free parameters via lenses — users can adjust source trust and re-render the entire knowledge graph in milliseconds. Agents are research workers (search/extract/audit), not voters. The mechanical containment check (`strings.Contains(source.body, citation.verbatim_quote)`) is the wall against LLM hallucination. See DESIGN.md for th
---

# CLAUDE.md — ground

## Project Overview

Ground is a source-anchored encyclopedia of weighted facts. Every claim traces back to verbatim quotes from cited sources. Sources carry credibility scores, exposed as free parameters via lenses — users can adjust source trust and re-render the entire knowledge graph in milliseconds. Agents are research workers (search/extract/audit), not voters. The mechanical containment check (`strings.Contains(source.body, citation.verbatim_quote)`) is the wall against LLM hallucination. See DESIGN.md for the full v2 specification.

The v1 design ("12 LLM personalities argue, EigenTrust extracts truth") is archived in git history under tag `v1-final`. Do not write new code against v1 concepts (assertions, reviews, support/contest stances, helpfulness ratings, contest quotas).

## Style Guide

This project follows the same conventions as ~/repos/wingthing:

- **Single binary Go + SQLite**. No Docker, no microservices, no ORM.
- **modernc.org/sqlite** (pure Go, no CGO). WAL mode + foreign keys enabled on open.
- **Embedded migrations** via `//go:embed migrations/*.sql`. Tracked in `schema_migrations` table. Auto-run on DB open.
- **Cobra CLI** in `cmd/ground/main.go`. Version injected via ldflags.
- **Go 1.22+ http.ServeMux** routing (`"GET /topic/{slug}"`). No framework.
- **Server-rendered HTML** via Go templates. No React, no npm, no node_modules. D3.js is the only JS dependency (for graph viz).
- **REST API** under `/api/` with JWT auth. Web UI and API served on the same port from the same binary.
- **stdlib `log`** only. No logging library.
- **`fmt.Errorf("context: %w", err)`** for all error wrapping. No panics.
- **No ORM**. Raw `sql.Query`/`QueryRow` with `Scan`. Helper functions for scanning rows.
- **Pointer fields** for nullable columns (`*string`, `*time.Time`).
- **Environment variables** for secrets and infrastructure config. No config file for MVP.

## Project Structure

```
ground/
├── CLAUDE.md
├── DESIGN.md
├── SKILLS.md          Bot developer / agent integration guide
├── TOPICS.md          Seed topic map with dependency structure
├── FACTS.md           Axiomatic nodes (adjudicated at seed time)
├── TODO.md
├── Makefile
├── README.md
├── go.mod / go.sum
├── cmd/ground/main.go       Cobra root + subcommands (server + client mode)
├── internal/
│   ├── db/                  SQLite open, migrations, query methods
│   │   └── migrations/      Numbered .sql files (001_init.sql, 002_v2_schema.sql, ...)
│   ├── model/               Data types (Agent, Topic, Claim, Source, Citation, Audit, Lens, Dependency, Epoch)
│   ├── sources/             Source fetching, body blob storage, content-hash dedup, citation graph extraction
│   ├── lens/                Per-request lens render: sparse merge, linear groundedness pass, topo DAG
│   ├── agent/               Seed orchestration — registers agents, launches claude -p
│   ├── client/              HTTP client for remote Ground instances (CLI client mode)
│   ├── engine/              Per-epoch baseline: source credibility, agent reliability, claim groundedness
│   ├── embed/               Embedding generation (OpenAI), cosine similarity, duplicate detection
│   ├── api/                 REST API handlers, JWT auth middleware, rate limiting
│   └── web/                 HTML handlers, template rendering, lens-aware views
├── prompts/                 Seed agent personality files (search/extraction strategy biases — NOT epistemic stances)
├── tasks/                   Seed round task descriptions (v2-search.md, v2-extract.md, v2-audit.md, v2-deps.md)
├── templates/               Go HTML templates
├── static/                  CSS, D3.js
└── ground.db                (gitignored)
```

## Key Concepts

- **Sources** are first-class. Every source has a fetched, cached body (content-addressed by sha256), metadata, tags, and a credibility score.
- **Citations** link a claim to a source via a verbatim quote that must literally appear in the cached body. The mechanical containment check is the wall against LLM hallucination — failing it returns 400 before any LLM judgment runs.
- **Audits** are agents verifying other agents' citations. Two stages: server-automatic mechanical re-check, then LLM-driven semantic verdict (`confirm`, `misquote`, `out_of_context`, `weak`, `broken_link`). Auditors cannot audit their own citations.
- **Agents** are research workers. Roles: `extractor`, `auditor`, `both`, `observer`, `admin`. Scored on **reliability** (audit-weighted citation pass rate) and **productivity** (capped throughput).
- **Claims** are atomic propositions with citations. Intrinsic groundedness is the credibility-weighted balance of supporting / contradicting / qualifying citations — linear in source credibility (this is what makes lens render cheap). Effective groundedness flows through a dependency DAG.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ehrlich-b/ground](https://github.com/ehrlich-b/ground) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
