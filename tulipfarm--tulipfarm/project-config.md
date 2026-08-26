---
trigger: always_on
description: Root entry point for AI coding agents. Read this, then the nearest `AGENTS.md` to your task —
---

# AGENTS.md

Root entry point for AI coding agents. Read this, then the nearest `AGENTS.md` to your task —
not the whole repo.

> **Terminology is binding.** Use the canonical names in
> [`metadata/terminologies.md`](metadata/terminologies.md) at every layer (code, DB, REST, URL,
> UI, docs). That file also lists banned synonyms and the rename backlog.

## What this is

TulipFarm is a **self-hosted control panel where autonomous agents run a business's
operations**. The user describes what they want in chat — "track our customers", "create a
support agent", "review this pull request" — and agents build and run it. Users never edit
files or write code to configure it. It runs on the operator's own infrastructure and model
provider keys; business data leaves the instance only when an agent was authorized to send it.

**That product promise is a constraint on you**: if a capability cannot be reached from chat or
the UI, it does not exist for users. Never close a gap by hand-editing the runtime `soul/` repo.

An instance has two halves:

- **The soul** — a git-backed config store (`soul/`, a separate repo, not a workspace). Resource
  schemas, agents, skills, routines and integrations live there as files. Agents write to it when
  asked to build something, so its git history is the audit trail.
- **The runtime** — the API and workers that load the soul, store records, index knowledge, and
  execute agent turns against configured LLM providers.

### The nouns you will meet in code

Full glossary with banned synonyms: [`metadata/terminologies.md`](metadata/terminologies.md).

- **Chat** is the external word (routes, URLs, UI); **Conversation** is the internal entity
  (table, repo, domain). Never let them bleed. A Conversation holds **Turns**, which hold
  **Messages**.
- **Agent** — a configured persona with its own instructions, tools and bounded authority.
- **Resource type** — a user-defined schema (Ticket, Customer); one instance is a **Record**.
  Never call an instance a "resource".
- **Routine** — a scheduled or triggered automation, built from **States**. One execution is a
  **Run**, which emits ordered **Run events**, each with an **Audience** (participant vs operator).
- **Integration** — a connected third party, fully defined by a declarative **manifest**:
  **egress** (what agents may do to the provider) and **ingress** (what the provider may send).
- **Skill** — an installable capability package. **Knowledge** — cited, ACL-preserving
  retrieval. **Memory** — scoped, versioned assertions. **Tool** — a callable an agent
  invokes, brokered with approvals. **Surface** — the channel-neutral protocol for
  rendering agent output.

### Stack

pnpm + Turborepo monorepo, TypeScript throughout.

- **Node** `26.5.0` (`.node-version`) · **pnpm** `11.5.3` — never npm/yarn
- **Workspaces**: `apps/*`, `packages/*`
- PostgreSQL (pgvector + pg-boss), Fastify API, Remix web UI

## Navigating this repo

Every app and package owns an `AGENTS.md` that states what it is, when to read it, and its
directory map. **Use them instead of grepping the repo.** `CLAUDE.md` files are pointers to the
sibling `AGENTS.md`.

| Path | Read it when your task touches |
| --- | --- |
| [`apps/api`](apps/api/AGENTS.md) | HTTP routes, auth/sessions, migrations, OpenAPI, soul git store |
| [`apps/web`](apps/web/AGENTS.md) | Remix UI, routes, loaders/actions, product screens |
| [`apps/worker`](apps/worker/AGENTS.md) | Run dispatch, Agent/Tool States, timers, reconciliation, projections |
| [`apps/integration-worker`](apps/integration-worker/AGENTS.md) | Integration ingress, sync, delivery, retries |
| [`apps/docs`](apps/docs/AGENTS.md) | Public Fumadocs site content and conventions |
| [`apps/eval`](apps/eval/AGENTS.md) | Offline eval Corpus, Expectations, Sweeps, Scorecards, red team |
| [`packages/agent-runtime`](packages/agent-runtime/AGENTS.md) | Context assembly, bounded Tool loop, model profiles, delegation |
| [`packages/run-kernel`](packages/run-kernel/AGENTS.md) | Run/State machines, waits, retries, child Runs |
| [`packages/files`](packages/files/AGENTS.md) | Upload limits, the type allowlist, magic-byte sniffing, the `files` table |
| [`packages/curator`](packages/curator/AGENTS.md) | Curator prompt, output schema, citation and injection validation, proposal templating |
| [`packages/curator-host`](packages/curator-host/AGENTS.md) | Minting a Curator job and its Run, context pinning, output revalidation, crash recovery |
| [`packages/model-adapter`](packages/model-adapter/AGENTS.md) | Translating `ModelPort` requests, tool calls and usage to and from the AI SDK |
| [`packages/turn-executor`](packages/turn-executor/AGENTS.md) | Chat Turn execution, Agent States, Turn guardrails, Run events |
| [`packages/tool-broker`](packages/tool-broker/AGENTS.md) | Tool catalog, intent/effect orchestration, approvals |
| [`packages/tool-host`](packages/tool-host/AGENTS.md) | Tool contract, authorization gate, dispatcher, co-location rule |
| [`packages/kv`](packages/kv/AGENTS.md) | Agent key-value store and its `kv_*` Tool family |
| [`packages/platform-tools`](packages/platform-tools/AGENTS.md) | Platform Tools that need no Soul, renderer or credential, so both hosts can run them |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TulipFarm/tulipfarm](https://github.com/TulipFarm/tulipfarm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
