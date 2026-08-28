---
trigger: always_on
description: enables it for localhost-class hosts — which is what you want behind a proxy; pass
---

# CLAUDE.md

Working guide for Claude Code in this repo. `AGENT.md` carries the same guidance for other
coding agents — **keep the two in sync when you change either.**

## What this repo is

A learning repo for **MCP protocol revision `2026-07-28`** (the stateless revision), built
twice: once with the Python SDK v2 (`mcp==2.0.0`) and once with the TypeScript SDK v2
(`@modelcontextprotocol/* @ 2.0.0`). It is teaching material — clarity beats cleverness, and
every example must actually run.

## Reference

**[REFERENCE.md](./REFERENCE.md) is the first thing to read.** It carries the `2026-07-28` wire
format (headers, `_meta` envelope, `resultType`, cache fields, the MRTR round trip), both SDK
surfaces with verified signatures, the deployment contracts, and a table mapping each failure
symptom to its cause. It was captured from the installed packages and running code, so it is
usually faster and more reliable than the spec site for anything these lessons touch. Each
lesson directory also has a README.

## Prerequisites

Node **≥ 20** (every `@modelcontextprotocol/*@2.0.0` package sets `engines.node: ">=20"`),
npm, [uv](https://docs.astral.sh/uv/), and `curl` for the `*.sh` scripts. Python does not need
installing separately — each lesson carries a `.python-version` pinning **3.13+** and
`uv sync` fetches a matching interpreter. Verified against Node 26.7, npm 11.19, uv 0.12.5.
Docker (with Compose) is needed for lessons `04-container` and `05-vercel` — verified on
Docker 29. Lessons `06-neon` and `07-rag` need a **Neon** connection string, and `07` also an
**embeddings API key**; both have free, card-free tiers, and each lesson README documents how
to obtain them. Secrets live in a git-ignored `server/.env` (copy the `.env.example` beside
it) and in the platform's env store when deploying — never in the repo. Lessons `00`–`05` need
no account of any kind. The **Vercel CLI** (`npm i -g vercel`, then `vercel login`; check with
`vercel whoami`) is needed only to *deploy* lesson `05`; both its images build and run locally
without an account. Otherwise no database, API key or network access beyond the two package registries;
lessons `00`–`03` bind `127.0.0.1` and everything runs unauthenticated.

## Commands

The two stacks are **lesson-for-lesson identical**: `00-helloworld`, `01-stateless`,
`02-pydantic`/`02-zod`, `03-mrtr`, `04-container`, `05-vercel`, `06-neon`, `07-rag`, each a
`server/` and a `client/` (except `02`, which runs both in one process; `05`–`07` add a `site/`). Both are **one project per lesson half** — a uv project in
Python, an npm package in TypeScript — installed and run where it lives. Neither `python/`
nor `typescript/` is itself a project, so there is no root install and no repo-wide test
command.

| lesson | Python (`cd python/<dir>`, `uv sync`) | TypeScript (`cd typescript/<dir>`, `npm install`) |
|---|---|---|
| 00 hello world | `00-helloworld/server` (:8000) + `00-helloworld/client` | `00-helloworld/server` (:3000) + `00-helloworld/client` |
| 01 stateless | `01-stateless/server` (:8001) + `01-stateless/client` | `01-stateless/server` (:3001) + `01-stateless/client` |
| 02 schemas | `02-pydantic` (no port) | `02-zod` (no port) |
| 03 MRTR | `03-mrtr/server` (:8003) + `03-mrtr/client` | `03-mrtr/server` (:3003) + `03-mrtr/client` |
| 04 containers | `04-container/server` (:8004) + `04-container/client` | `04-container/server` (:3004) + `04-container/client` |
| 05 Vercel | `05-vercel/server` (:8005 local, :80 in image) | `05-vercel/server` (:3005 local, :80 in image) |
| 06 Neon | `06-neon/server` (:8006) + `06-neon/client` | `06-neon/server` (:3006) + `06-neon/client` |
| 07 RAG | `07-rag/server` (:8007) + `07-rag/client` | `07-rag/server` (:3007) + `07-rag/client` |
| run | `uv run python main.py` | `npm start` |
| tests | `uv run pytest` (server projects) | `npm test` (server packages) |
| curl | `./hello.sh` / `./call.sh` / `./deploy.sh` | `npm run hello` / `call` / `deploy` |

TypeScript packages also answer to `npm run dev`, `npm run typecheck` and
`npm run build`/`npm run serve`.

Lesson `04` adds `docker build` / `docker compose up --build` (three replicas behind nginx).
Lesson `05` builds `Dockerfile.vercel` and deploys with
`vercel deploy --yes --scope <team>` **from the lesson root** (the directory holding
`vercel.json`, not `server/`).
Docker is the only extra prerequisite; the Vercel CLI is needed only to deploy or run
`vercel dev`, and neither lesson requires an account to build and run the image locally.

Servers and clients are separate processes: start the server first, in its own terminal.

## Non-negotiable rule: verify before you write

This revision and both SDK v2 lines post-date most model training data, and v2 renamed a lot.
**Do not write MCP code from memory.** Before using an API:

- introspect the installed package (`uv run python -c "import inspect; ..."`, or read
  `node_modules/@modelcontextprotocol/*/dist/*.d.mts`), or
- check the spec at <https://modelcontextprotocol.io/specification/2026-07-28>,

then **run the example** and paste real output. An example that has not been executed does not
get committed.

## Verified facts worth not rediscovering

These were each confirmed by running code in this repo.

**Python SDK v2**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [panaversity/learn-stateless-mcp](https://github.com/panaversity/learn-stateless-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
