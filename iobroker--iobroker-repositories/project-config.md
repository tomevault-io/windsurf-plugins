---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

Two data files plus the automation that guards them. The data files *are* the product — they are
served to every ioBroker installation as the adapter repository:

- `sources-dist.json` — the **latest** (beta) repository, ~800 adapters.
- `sources-dist-stable.json` — the **stable** repository, a subset of latest pinned to a version.

Everything in `src/` and `lib/` exists to validate, mutate or publish those two files. Most of it runs
as a GitHub Action against a pull request, not on a developer machine.

The code is split **by author**, on purpose - do not migrate one side into the other:

- `src/*.mts` - Bluefox's scripts (`scripts`, `tools`, `build`, `common`, `check`, `test/testRepo`).
  TypeScript, executed **directly** by Node's built-in type stripping: `node src/<x>.mts`. Nothing is
  ever compiled - there is no `build/`, no `prepare` step, and `tsc` is only the type checker
  (`npm run typecheck`). This needs Node >= 22.19 (`engines` in package.json).
- `lib/*.js` + `test/checkRepository.test.js` - Martin's (mcm1957) workflow scripts. Plain CommonJS
  JavaScript, run directly (`node lib/<x>.js`), kept in their original style: not type-checked, not
  linted. Their only link to the TypeScript side is `require('../src/common.mts')` for the GitHub API
  wrappers - Node loads that ES module through `require()`, so the same Node >= 22.19 applies to them.

## Commands

```bash
npm i                 # dependencies only - there is no build step
npm run typecheck     # tsc -p tsconfig.json (noEmit); run it before committing changes to src/
npm run lint          # eslint . - covers src/ only, lib/ and test/ are ignored on purpose

# Mutating the repo files — always go through these, never hand-edit the JSON
npm run addToLatest  -- --name <adapter> --type <type>      # discovers GitHub repo/branch/icon itself
npm run addToStable  -- --name <adapter> [--version x.y.z]  # must already be in latest + on npm
npm run updateStable -- --name <adapter> [--version x.y.z]  # bump an existing stable pin
npm run sort                                                # re-sort + re-normalize both files
npm run nodates                                             # strip versionTime/versionDate

# Tests
npm test                                                   # mocha over src/test/*.mts + test/*.js — SLOW (see below)
npx mocha test/checkRepository.test.js --exit              # offline unit tests only, ~instant
npx mocha src/test/testRepo.mts --exit --grep "reserved"   # one case from the network suite

# Structural validators — each is a standalone CLI that exits non-zero on failure (plain JS, no dependencies)
node lib/checkRepository_checkJsonFormatting.js sources-dist.json
node lib/checkRepository_checkLatestAttributes.js
node lib/checkRepository_checkAdapterRepositoryFiles.js   # network; wants OWN_GITHUB_TOKEN
```

`npm test` loads `src/test/testRepo.mts`, which fetches every `meta` URL in both files with a 1 s delay
between requests — it takes 15+ minutes and needs `OWN_GITHUB_TOKEN` to avoid rate limits. Prefer the
single-file / `--grep` forms while iterating.

`npm run check` only works inside a GitHub Action: it reads `GITHUB_REF` / `GITHUB_EVENT_PATH` to find
the PR number and posts comments back. Each such script has a commented-out block near the bottom
(`// process.env.GITHUB_REF = ...`, `event.json`) that is the intended way to run it locally; `event.json`
is gitignored for that purpose. Without those variables the scripts abort with "Reference not found"
before touching the network, which makes `node src/check.mts` a safe smoke test.

## TypeScript layout

- `tsconfig.json` is the single source of truth and is **type-check only** (`noEmit`). `module`/
  `moduleResolution` are `nodenext`, `allowImportingTsExtensions` permits the `.mts` extensions Node
  needs in relative imports, `erasableSyntaxOnly` rejects anything type stripping cannot erase (`enum`,
  `namespace`, parameter properties), and `verbatimModuleSyntax` forces explicit `import type` so no
  stripped import ends up referencing a type-only export at runtime.
- The sources are `.mts`, not `.ts`, on purpose: package.json must not get `"type": "module"` (Martin's
  `lib/*.js` are CommonJS), and a plain `.ts` with `import`/`export` is then parsed twice by Node with a
  `MODULE_TYPELESS_PACKAGE_JSON` warning. `.mts` is unconditionally an ES module. Consequences inside
  `src/`: `import.meta.dirname` / `import.meta.filename` instead of `__dirname`, no bare `require`
  (`createRequire(import.meta.url)` where a CommonJS package must be loaded that way), and relative
  imports carry the `.mts` extension.
- Every workflow runs `npm i` before its script, for dependencies only. The nine `checkRepository.yml`
  jobs skip even that — those scripts have no dependencies at all. `node-version: 22` in the workflows
  resolves to the newest 22.x, which satisfies the >= 22.19 requirement.
- `@iobroker/eslint-config` only targets `**/*.ts`; `eslint.config.mjs` widens every such block to

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ioBroker/ioBroker.repositories](https://github.com/ioBroker/ioBroker.repositories) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
