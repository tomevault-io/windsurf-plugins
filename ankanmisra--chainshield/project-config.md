---
trigger: always_on
description: Project conventions and pointers for AI coding agents (and humans) working in this repo.
---

# AGENTS.md

Project conventions and pointers for AI coding agents (and humans) working in this repo.

## What this project is

ChainShield Agent: a policy-bound risk gate for onchain treasuries and wallets. The server accepts a transaction intent, evaluates it against deterministic rules plus (eventually) simulation and LLM reflection, and returns one of three verdicts: `ALLOW`, `REQUIRE_HUMAN_CONFIRMATION`, or `BLOCK`. Every decision is appended to an incident timeline.

Built for ETHGlobal OpenAgents 2026. Sponsor integrations planned: 0G (storage + inference), KeeperHub (remediation playbooks), Gensyn AXL (multi-agent mesh).

## Tech stack

This hackathon submission is **TypeScript on Bun, end to end**. There is no Rust code and no Solidity contracts in this repo. A Rust + Solidity port lives in the post-hackathon roadmap (see `docs/architecture.md` "Future direction" section) but no agent should write `.rs` or `.sol` files in this repo without an explicit user request.

- Runtime: Bun 1.3 (single tool for install, run, test, build)
- Language: TypeScript with strict mode (`noUncheckedIndexedAccess`, ESM with `.js` import suffix)
- HTTP: Fastify 5 + `@fastify/cors`
- Validation: Zod (boundary only — internal code trusts typed values)
- Onchain client: `ethers` v6 (peer dep of the 0G storage SDK)
- 0G persistence: `@0gfoundation/0g-storage-ts-sdk`
- Frontend: Astro 6 at `web/` (vanilla TS, no React/Vue)
- Tests: `bun:test` (Jest-compatible)
- Containerization: Docker (multi-stage on `oven/bun:1.3-alpine`)

## Run, test, build

```sh
bun install                # install deps from bun.lock
bun run dev                # start risk-gate server with watch on 127.0.0.1:8787
bun run start              # production-style boot (no watch)
bun test                   # 114 specs across 13 files
bun run test:coverage      # v8 coverage report
bun run typecheck          # tsc --noEmit, must exit 0
bun run build              # bundle to ./dist/server.js
bun run start:bundle       # run the bundled output
bun run clean              # remove dist/, coverage/, .tsbuildinfo
bun run demo               # four-scene CLI runner against the live server
```

Docker path:

```sh
docker compose up --build  # builds and runs on host port 8787
```

## Where things live

- `src/core/` — types, Zod schemas, policy service, decision engine, EVM selector helpers
- `src/memory/` — `Store` interface, `InMemoryStore`, `ZeroGStore` (0G anchor adapter)
- `src/simulator/` — `Simulator` interface, `HeuristicSimulator` (ERC-20 calldata decode + balance projection)
- `src/playbooks/` — `PlaybookRunner` interface, `KeeperHubRunner`, notification channels
- `src/risk-gate/` — Fastify app and server entrypoint
- `src/cli/demo.ts` — `bun run demo` four-scene CLI
- `tests/` — `bun:test` specs covering engine rules, policy service, the API, simulator, playbooks, 0G store, anchor surfacing
- `web/` — Astro 6 frontend (components, lib modules, pages, styles)
- `docs/` — architecture, submission, demo script, product story, sponsor research notes
- `scripts/` — `kh.sh` (KeeperHub helper), `dev.sh` (parallel server + frontend dev)
- `.claude/skills/` — reusable agent skills for repeated tasks. Note: `rust-backend-style/`, `solidity-contracts/`, and `sponsor-wiring/` describe the post-hackathon Rust port, not the current code.

## Conventions (TypeScript)

### TypeScript code style

- Strict TypeScript. `noUncheckedIndexedAccess` is on, so always handle `undefined` from `arr[i]` and `Map.get`.
- ESM only. Imports of local files use the `.js` extension (Node ESM convention; Bun and `tsc` both honor it).
- No default exports for app code.
- Prefer interfaces for public shapes, type aliases for unions and primitives.
- Validate at the boundary (HTTP body, env, untrusted JSON) with Zod, then trust the typed value internally.
- Keep error messages user-facing; the Fastify error handler already formats Zod errors as 400s.
- One interface per behaviour seam (`Store`, `Simulator`, `PlaybookRunner`, `NotificationChannel`); keep concrete adapters substitutable.

### Tests

- Co-located test data in `tests/helpers.ts` (canonical addresses, intent and policy factories).
- One feature per `it()`. Assertions on the verdict, the matched rules, and the risk score together; do not assert just one of them.
- Inject `now()` and `idGen()` into `DecisionEngine` and `PolicyService` for deterministic timestamps and ids.
- Do not mock the in-memory store; use a fresh `InMemoryStore` per test.

### Commits

- One-line subjects when possible. Imperative mood (`add`, `fix`, `refactor`, never `added`/`adding`).
- Lowercase first letter unless it is a proper noun.
- Group related changes into a single commit; split unrelated work.
- Never amend a pushed commit unless the user asks for it.

### Branches

- Feature branches: `feature/<short-name>`.
- Default base for PRs: `main`.
- CI on every PR and push to `main` runs `.github/workflows/ci.yml`: frozen-lockfile installs (root + `web/`), `bun run typecheck`, `bun test`, `bun run build:web`, and an emoji-bytes scan. The Bun version is pinned in `.bun-version`.

## Decision-engine contract (do not break)

The engine is the heart of the product. Keep these invariants when changing it:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AnkanMisra/ChainShield](https://github.com/AnkanMisra/ChainShield) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
