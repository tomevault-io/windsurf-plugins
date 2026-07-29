---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`@oceanprotocol/lib` (currently v8.x) — the JavaScript/TypeScript client library for Ocean Protocol. It lets apps publish data as ERC721 data NFTs + ERC20 datatokens, price them (fixed-rate exchange, free dispenser), and consume them (download or Compute-to-Data), talking to on-chain contracts via ethers and to off-chain Ocean Node services (Provider / Aquarius) over HTTP or libp2p P2P.

- Pure ESM package: `"type": "module"`, entry `src/index.ts`, Node `>=18` (CI builds on 20 & 22; `.nvmrc` = 22).
- Blockchain layer is **ethers v6** (the `web3` peerDependency is legacy and not used by the wrappers).
- Ships bundled by microbundle to `dist/` (cjs / esm / modern / umd + `dist/types/*.d.ts`).
- DDO parsing/validation and DID versioning are delegated to the external `@oceanprotocol/ddo-js` (`DDOManager`); ABIs come from `@oceanprotocol/contracts`.

## Common commands

Build / develop:
- `npm start` — clean, generate metadata, then `tsc -w` (watch mode).
- `npm run build` — production build: `clean` → `build:metadata` → microbundle (formats modern,esm,cjs,umd, compressed). This is what `prepublishOnly` runs.
- `npm run build:tsc` — plain `tsc --sourceMap` (no bundling).
- `npm run build:metadata` — writes `src/metadata.json` (package version + git commit) via `scripts/get-metadata.js`. **Required before type-check and before running tests**; `type-check`, the release hook, and CI all invoke it. If you see stale/odd metadata errors, re-run this.
- `npm run clean` — removes `dist/`, `doc/`, `.nyc_output`.

Lint / format / types:
- `npm run lint` — ESLint over `.ts,.tsx` **and** `npm run type-check`.
- `npm run lint:fix` — ESLint autofix.
- `npm run format` — Prettier over `**/*.{js,jsx,ts,tsx}`.
- `npm run type-check` — `build:metadata` + `tsc --noEmit`.
- Style is enforced by `.prettierrc` (no semicolons, single quotes, `printWidth` 90, no trailing commas, 2-space) and `.eslintrc` (extends `oceanprotocol` + `plugin:prettier/recommended`; empty catch allowed; several rules downgraded to `warn`).

Docs (generated — do not hand-edit output):
- `npm run docs` — TypeDoc + `typedoc-plugin-markdown` → `docs/` (per-class/interface markdown).
- `npm run doc:json` — TypeDoc JSON → `dist/lib.json` (attached to GitHub releases).

Tests:
- `npm test` — the full CI-equivalent gate: `lint` → `test:unit:cover` → `test:integration:cover`.
- `npm run test:unit` — unit tests only: `npx tsx ./node_modules/mocha/bin/mocha.js --config ./test/.mocharc.json 'test/unit/**/*.test.ts'`.
- `npm run test:unit:cover` — same under `nyc` (coverage → `coverage/unit`).
- `npm run test:integration` — integration tests, **excluding** `Sapphire.test.ts`.
- `npm run test:integration:cover` — same under `nyc` (coverage → `coverage/integration`, `--no-clean` so it merges with unit).
- `npm run test:sapphire` — runs only `test/integration/Sapphire.test.ts` against the **live Oasis Sapphire testnet** (needs `PRIVATE_KEY` and `PRIVATE_KEY_CONSUMER` exported); kept out of the normal integration run.
- `npm run mocha` — raw mocha runner (`TS_NODE_PROJECT=./test/tsconfig.json mocha --config=test/.mocharc.json --node-env=test --exit`); this is what `test:sapphire` wraps.

Running a SINGLE test:
- One file (recommended, matches the maintained unit/integration scripts):
  `npx tsx ./node_modules/mocha/bin/mocha.js --config ./test/.mocharc.json 'test/unit/Datatoken.test.ts'`
- A single `it(...)` case: add `--grep 'partial test title'` to the command above.
- The repo's own single-file pattern (used by `test:sapphire`) is `npm run mocha -- 'test/integration/<File>.test.ts'`.
- Note: both mocharc files set `bail: true`, so a run stops at the first failing assertion. Timeouts differ: root `.mocharc.json` = 200000ms, `test/.mocharc.json` (used by the npm scripts) = 20000ms.

### Unit vs integration split

- `test/unit/**` — pure contract-wrapper tests (Datatoken, Nft, NftFactory, FixedRateExchange, Dispenser, Router, Escrow, EnterpriseFeeCollector, AssetUtils). These still hit a **local chain** (they deploy/interact via the barge Ganache node) but don't require the full Node/Aquarius/Indexer stack the way integration does. `mock-local-storage` and `source-map-support` are auto-required (mocharc).
- `test/integration/**` — end-to-end flows against the whole Ocean stack: `PublishFlows`, `PublishEditConsume`, `ComputeFlow`, `Provider`, `Auth`, `Sapphire`, plus `CodeExamples`/`ComputeExamples` (see "Generated guides"). `_P2PWarmup.test.ts` warms up the libp2p transport first.
- Shared test setup is in `test/config.ts` (`getTestConfig`, `getAddresses`, a default `JsonRpcProvider` on the development network) and `test/integration/helpers.ts`.

### Integration tests need Barge (local Ocean stack)

Integration tests run against [Barge](https://github.com/oceanprotocol/barge), which spins up a local blockchain (chainId `8996`, network name `development`), an Ocean Node (Provider + Aquarius + Indexer), and an Elasticsearch DB in Docker.

```bash
git clone https://github.com/oceanprotocol/barge && cd barge
./start_ocean.sh
# in another terminal, back in ocean.js:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oceanprotocol/ocean.js](https://github.com/oceanprotocol/ocean.js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
