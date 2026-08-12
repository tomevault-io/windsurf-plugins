---
trigger: always_on
description: > **This is the canonical development guide for LoreWeave — for humans and for AI coding agents alike.**
---

# AGENTS.md — LoreWeave Development Guide

> **This is the canonical development guide for LoreWeave — for humans and for AI coding agents alike.**
> It is tool-neutral. `CLAUDE.md` at the repo root is a pointer to this file, and any other
> assistant-specific entry point (`.cursor/rules`, `.github/copilot-instructions.md`, …) should point
> here too rather than fork the content. **One home, one name** — the same rule this repo applies to
> settings applies to its own guidance.
>
> **New contributor? Read [`CONTRIBUTING.md`](CONTRIBUTING.md) first** — it is the short entry point
> (setup, where to find things, how to open a PR). This file is the *deep* rulebook: the invariants,
> the standards, and the hard-won bug lore behind them. Most rules below exist because the described
> failure actually shipped here. **Do not treat them as style preferences.**
>
> A handful of sections reference harness features of a specific agent CLI (`/goal`, `/compact`).
> Those are marked where they appear; the *discipline* they encode is tool-neutral
> and applies however you work.

> **Your agent framework is a runner, not a rulebook.** This repo commits the `aif-*` skill pack
> ([AI Factory](https://github.com/lee-to/ai-factory), MIT), installed once per agent target —
> `.claude/`, `.cursor/`, `.codex/`, `.agents/` (Codex app), `.github/skills/` (Copilot) — plus
> `.ai-factory/`. Those decide *how* an agent moves through work; they never decide what this
> project permits. **Where a skill's default conflicts with this file, this file wins** — and that conflict
> is a defect to report, not a judgement call to make mid-task. (Live example: `aif-implement` says
> *"do not add tests by default"*, which is wrong here, and is overridden.)
>
> Project rules are bound into those skills at `.ai-factory/skill-context/<skill>/SKILL.md` — the
> pack's own sanctioned override hook, which it treats as a mandatory read that beats its defaults.
> **Those skill trees are GENERATED, one per agent target; never hand-edit one
> copy** (`scripts/agent-skills-parity.py` blocks it at pre-commit). How to contribute to the
> workflow, and how to personalise it without changing it for everyone else:
> [`docs/standards/agent-workflow.md`](docs/standards/agent-workflow.md).

## What This Project Is

LoreWeave is a multi-agent platform for multilingual novel workflows (translation, analysis, knowledge building, assisted creation). Cloud-hosted (AWS) monorepo with Docker Compose for local development. Serves multiple users across multiple devices (PC, mobile, tablet).

Source of truth for current status: `docs/sessions/SESSION_HANDOFF.md`

---

## Architecture

Monorepo layout. This lists the **top-level trees**, which are stable; it deliberately does not
enumerate their contents (see Services below for why an inlined service table is a rot hazard).
- `services/` — microservices (Go/Chi, Python/FastAPI, TypeScript/NestJS)
- `crates/` — shared Rust libraries and the simulation/domain kernels
- `pkg/` — shared Go packages
- `sdks/` — reusable language SDKs · `clients/` — typed service clients
- `packages/` — shared TypeScript packages
- `frontend/` — Vite + React + Tailwind + shadcn/ui (novel workflows)
- `frontend-game/` — living-world game client · `cms-frontend/` — admin client
- `contracts/` — OpenAPI, event, schema, registry, and invariant sources of truth
- `migrations/` — repository-level migration assets
- `docs/` — governance, planning, and session docs · `runbooks/` — operational response
- `scripts/` — validation, generation, and operational tooling (the gates live here)
- `infra/` — docker-compose and infra config

### Services

The repo has **47 services** (verify: `ls services/ | wc -l`). This file does **not** enumerate them — the old inlined table went stale and misled agents. **Authoritative service→language map: [`contracts/language-rule.yaml`](contracts/language-rule.yaml); purposes: [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md); frontend feature → route → backing service: [`docs/FEATURE_INDEX.md`](docs/FEATURE_INDEX.md); data layer: [`docs/DATA_ARCHITECTURE.md`](docs/DATA_ARCHITECTURE.md).** Language rule: Go = domain/meta · Python = AI/LLM · TS = gateway/realtime · Rust = kernel-derived. Do not assume a service is absent because it's not named here.

Load-bearing facts an agent needs regardless:
- **`provider-registry-service`** is the ONLY home of provider SDKs/keys (Provider-gateway invariant below).
- **`glossary-service`** = authored SSOT for lore AND hosts the **wiki** (`wiki_*` tables — wiki is NOT a separate service). **`knowledge-service`** = derived fuzzy/semantic layer (Postgres SSOT + Neo4j) anchored to glossary via `glossary_entity_id` FK (two-layer pattern; see [scope-separation](docs/standards/scope-separation.md)).
- **`api-gateway-bff`** (TS/NestJS) = external entry point; **`chat`/`translation`/`knowledge`** = Python AI services; **`auth`/`book`/`sharing`/`catalog`/`usage-billing`** = Go domain.
- Data: Postgres (per-service DBs), RabbitMQ (job/event bus + outbox), Redis (cache/rate-limit), MinIO (objects).

### Key Rules

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [letuhao/lore-weave](https://github.com/letuhao/lore-weave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
