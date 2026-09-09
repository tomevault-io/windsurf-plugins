---
trigger: always_on
description: provides this automatically.
---

# AGENTS.md

Instructions for AI agents and contributors working in the `@alpacahq/alpaca-trade-api`
package.

## Overview

`@alpacahq/alpaca-trade-api` is a TypeScript SDK for the Alpaca **Trading API**
and **Market Data API**. The REST clients/models are generated with OpenAPI
Generator via the **reproducible pipeline in `tooling/`** (`npm run generate`);
they stay a faithful snapshot of Alpaca's OpenAPI spec, and every convenience is
hand-written in separate modules (see the first convention below). Crucially,
the generated trees are **never hand-edited** — every required deviation from
stock generator output is encoded declaratively in forked Mustache templates or
JSON Patch overlays, so regeneration reproduces the committed trees byte-for-byte.
See `tooling/GENERATION.md` for the full design. Notable behaviors to preserve
when editing:

- null-safe array deserialization (no NPE on `null` array fields),
- opt-in retry/backoff (`retry`), request timeouts (`timeoutMs`), default
  `User-Agent`,
- typed `ApiError` parsing the `{ code, message }` envelope,
- undocumented-field passthrough on key trading models,
- a `pagination` helper, and a `vitest` test suite.

## Conventions

- **Generated vs hand-written — never hand-edit the generated trees.** The
  `src/trading/{apis,models,index.ts}` and `src/market-data/{apis,models,index.ts}`
  trees are generator output, reproduced by `npm run generate` (see `tooling/`).
  Treat them as derived artifacts: **never hand-edit them**. If you need to change
  generated output, change the spec/overlay/template in `tooling/` and regenerate
  — `npm run generate:offline` must reproduce the trees byte-for-byte. All
  behavior, ergonomics, and fixes live in hand-written modules outside those trees
  (`src/client.ts`, `src/orders.ts`, `src/marketDataShapes.ts`,
  `src/core/runtime.ts`, `src/streaming/`, ...).
- **Regeneration-safe customizations live in `tooling/`.** Three deviations from
  stock `typescript-fetch` are encoded declaratively so they survive every
  regeneration: (1) null-safe required-array deserialization — forked
  `templates/typescript-fetch/modelGeneric.mustache`; (2) undocumented-field
  passthrough on 6 trading models — `x-ts-passthrough` vendor extension (trading
  overlay) + forked templates; (3) market-data `feed` enum tightening — market-data
  overlay. Add new fixes the same way (template or overlay), never as a hand-edit.
- **The transport is shared.** The HTTP transport (retry/backoff, timeouts,
 rate limiting, typed errors, middleware, querystring, response wrappers) lives
 once in `src/core/runtime.ts`. `src/trading/runtime.ts` and
 `src/market-data/runtime.ts` are thin shims that `export *` from it and only
 add their host constants plus a `Configuration` subclass overriding
 `defaultBasePath()`. Make transport changes in `src/core/runtime.ts`; touch the
 shims only for host/base-path concerns. These shims sit inside the generated
 trees but are hand-maintained transport code, protected from regeneration by
 `.openapi-generator-ignore`; treat them as the hand-written exceptions inside
 those otherwise generator-owned trees.
- **Edit `src/` directly** for behavior changes.
- **Keep the capability maps in sync.** When you add an ergonomic helper to
  `TradingClient` / `MarketDataClient` / `OrdersApi` (`src/client.ts`), add it to
  `ergonomicCapabilities` in `src/capabilities.ts` — a test in
  `test/client.test.ts` asserts every listed helper exists on the facade.
- **Linting is scoped to hand-written code.** Biome (linter only; formatter and
  assist are off) lints the hand-maintained TypeScript. The OpenAPI-generated
  `src/trading/{apis,models,index.ts}` and `src/market-data/{apis,models,index.ts}`
  are excluded in `biome.json` — they're generator output, so don't lint or
  hand-edit them.
- Keep the test suite green and add coverage for new behavior.

## Commands

Repository development, documentation, generation, and release tooling require
Node.js 24 (see `.nvmrc`). The published SDK remains compatible with Node.js 20
and CI verifies the packed package on that minimum version.

```bash
npm install            # also builds via the `prepare` script
npm run build          # tsup -> dist/ (dual ESM + CJS)
npm run typecheck      # tsc --noEmit (the type authority)
npm test               # vitest
npm run lint           # biome lint (hand-written code; generated apis/models are ignored)
npm run lint:fix       # biome lint --write (apply safe autofixes)
npm run generate       # regenerate REST trees: fetch latest specs, diff, confirm, generate
npm run generate:offline  # reproduce the trees from pinned specs (no network)
```

Regeneration lives in `tooling/` (a separate private package with its own deps
and tests). It needs a real JDK (auto-detected; `brew install openjdk` if
missing). See `tooling/GENERATION.md` for the pipeline, durability mechanisms,
and how to adopt upstream spec changes.

## Releases & changelog (Changesets)

Releases and `CHANGELOG.md` are managed with [Changesets](https://github.com/changesets/changesets).
Changelog entries are **human-authored**, decoupled from commit messages — every
user-facing change ships with a changeset file describing it and its semver bump.

Stable releases are published from `master` to the npm **`latest`** dist-tag.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alpacahq/alpaca-trade-api-js](https://github.com/alpacahq/alpaca-trade-api-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
