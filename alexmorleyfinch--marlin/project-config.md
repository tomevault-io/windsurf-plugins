---
trigger: always_on
description: This is a fully vibe coded project with oversight, so this file reflects the current truth. When the humans request contradicts this file, clarify the intent to stray from the definition.
---

# AGENT.md

This is a fully vibe coded project with oversight, so this file reflects the current truth. When the humans request contradicts this file, clarify the intent to stray from the definition.

Context for Cursor and future agents working in this repo. Human operator guide: `docs/GETTING_STARTED.md` (entities, config, CLI, LM Studio, vLLM/Vast). Policy inventory: `data/README.md`. Schema: `docs/MIGRATIONS.md`. Do not duplicate those step-by-steps here; keep this file to facts that are expensive to infer.

## What this is

Marlin is a **personal, single-user** search index aimed at tens of millions of domains. No auth, no multi-tenancy. Do **not** add `userId` or a domain-ignore table unless asked. Ignore is a boolean on `categories` and `tags`.

v1 discovery is a **domain list file** (ingest) plus **link following after LM** (outbound enqueue). An optional BFS **spider** can also expand from a few seeds into `pending` — not required for normal use. There is no IPv4/ICMP/TLS-SAN scanner.

## Layout

| Path | Owns |
| --- | --- |
| `apps/spider` | **Ingest** CLI (`src/ingest.ts`: domain list → `pending`) + optional **BFS spider** (`src/index.ts`: seed link-follow → `pending`). Spider is not the main crawl path — prefer ingest + fetcher; primary link growth is LM outbound enqueue after `done` |
| `apps/fetcher` | High-concurrency homepage fetch → store extracted text + outbound hosts (no enqueue) |
| `apps/worker` | Claim `ready` pages: near-empty / bot-check → `empty` (no LM), for-sale lander → `parked` (no LM), else one OpenAI-compatible LM call; `src/probe.ts` is the no-DB smoke test |
| `apps/steward` | Spiral detector: SQL-nominate busy apexes → LM sample judge → auto-block in Postgres; same `WORKER_PROFILE` as catalog worker; policy from `STEWARD_POLICY` / `--policy` |
| `apps/api` | Fastify `/api/*` search (incl. country/language, `{ hits, hasMore }`), ignore toggles, `/api/dashboard`, `/api/workers`, `/api/analyze/*` catalog analysis + label merge + steward unblock |
| `apps/web` | Vite + React search UI (query-string filters + load more), `/dashboard`, `/workers`, `/analyze` (overview / labels / platforms / steward), ignore modal |
| `packages/db` | Drizzle schema, SQL migrations, pool, queries, analyze aggregates, label-merge lib, migrate/requeue/flush-queue/merge-labels CLIs |
| `packages/shared` | Hostname normalize, TLD whitelist (`data/tlds.txt`), ICANN apex + subdomain cap, category/language crawl priority, fetch/extract, LLM JSON schema + catalog/steward policy loaders, geo normalize, `pickSiteName`, lm + worker profiles |
| `data/` | Forkable policy — see `data/README.md` |
| `data/domains.sample.txt` | Tiny ingest file for test runs |
| `data/seeds.makers.txt` | Maker / small-web seed hosts (ingest to bias discovery) |
| `data/blocked-apex.txt` | Seed crawler-trap apex denylist (Forumotion, B2B mills) — bootstrapped into `blocked_apexes` |
| `data/allowed-apex.txt` | UGC / platform apexes the steward must never auto-block |
| `data/category-priority.txt` | Per-category crawl/LM queue weights (edit + restart fetcher/worker) |
| `data/language-priority.txt` | Language demotion weights on outbound enqueue (`en` / `mul` / `default`) |
| `data/tlds.txt` | English-oriented last-label TLD whitelist |
| `data/lm-profiles.json` | Named LM connections (`baseUrl` / `model` / `apiKey` / `timeoutMs`). Used by probe/compare via `--lm`, and by worker profiles via `lm` key |
| `data/worker-profiles.json` | Named worker bundles (`lm` key + `concurrency`). Selected by `WORKER_PROFILE` or `npm run worker -- <name>` |
| `data/catalog-policies.json` | Catalog LM policy (prompt, `textChars`, sampling). Selected by `CATALOG_POLICY` or `--policy` on worker / probe / compare |
| `data/steward-policies.json` | Steward spiral-judge policy (prompt, `sampleSummaryChars`, sampling). Selected by `STEWARD_POLICY` or `--policy` on steward |
| `data/label-aliases.txt` | Tag/category spelling aliases — rewrite at `completeDomain`; CLI/UI merge for rows already in DB |

`packages/db` is the only place schema/SQL should live. `packages/shared` is the only place hostname rules, crawl-priority weights, and the LLM **schema** should live — spider/fetcher/worker/steward must not fork copies. Catalog/steward **prompts and sampling** live in `data/*-policies.json`.

Runtime is TypeScript via `tsx` (dev and Docker). Workspace `exports` point at `src/*.ts`.

## Invariants (do not “simplify” away)

- **Fetch and LM are separate processes.** Fetcher saturates the network; LM worker keeps profile `concurrency` in-flight LM calls with no sleep between successes. Do not merge them back into one sequential job — GPU idle time during HTTP is the whole point of the split.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alexmorleyfinch/marlin](https://github.com/alexmorleyfinch/marlin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
