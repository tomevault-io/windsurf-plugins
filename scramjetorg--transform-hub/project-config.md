---
trigger: always_on
description: - Read `codemap.md` before code changes; it lists the real entrypoints and package responsibilities.
---

# Agent Notes

## First reads
- Read `codemap.md` before code changes; it lists the real entrypoints and package responsibilities.
- For deep work in a package, read that package's `codemap.md` when present.

## Package manager
- Use `npm`, not `yarn`, for agent-run commands in this repo.
- A root `package-lock.json` exists, but many historical scripts/CI snippets still mention `yarn`; do not copy those blindly. Prefer `npm ci`, `npm install`, `npm run <script>`, or the underlying `node scripts/...` command.

## High-value commands
- Install deps: `npm ci` for a clean install, `npm install` when updating the lockfile.
- Build packages only: `npm run build:packages` (`scripts/build-all.js -v -w packages --ts-config tsconfig.build.json`).
- Full build is expensive: `npm run build` includes packages and Docker builds.
- Unit/package tests: `npm run test:packages` is the local default — `scripts/run-script.js -w packages -j 4 test` (four package test processes in parallel) with a 3.5 GiB aggregate-RSS budget. GitHub workflows use `npm run test:packages:ci` (`scripts/run-script.js -w packages -j 2 test`) with a 2.5 GiB aggregate-RSS budget. These are aggregate-RSS budgets, not `ulimit -v` caps: AVA 8 worker-thread isolates reserve large virtual address spaces, so a virtual-memory ulimit is unsuitable as the concurrency bound. `npm run test:packages:phase-final` remains the serial proof path.
- BDD smoke paths: `npm run test:bdd-ci-api-node`, `npm run test:bdd-ci-node`, `npm run test:bdd-ci-python`, or `npm run test:bdd`.
- Biome lint/format: `npm run lint`, `npm run lint:quick`, `npm run lint:fix`, `npm run format`, or the lower-level `npm run biome:check`/`npm run biome:lint`/`npm run biome:format` scripts. `lint` runs Biome linting; formatting remains an explicit `format` operation to avoid broad format churn.
- Biome scripts set `RAYON_NUM_THREADS=12` by default. This passed on the 24-core agent host under the repo virtual-memory cap with ~98 MB max RSS, while 24/default parallelism failed from native allocation pressure; do not silently raise the cap.
- Runtime invariant check: `npm run check:runtime-invariants`.
- Dev hub: `npm run start:dev`; built hub: `npm run start` after building `dist/`.

## Monorepo wiring
- Workspaces are explicit `packages/*` listed entries plus `bdd/`; custom workspace groups in `package.json` include `packages`, `release`, `runners`, and `bdd`.
- `scripts/run-script.js` runs a package script across workspaces; it defaults to 16 concurrent jobs (override with `-j <jobs>`). It runs every selected package after failures, then exits nonzero with aggregated failures; use `--fail-fast` or `SCRAMJET_RUN_SCRIPT_FAIL_FAST=1` to stop scheduling after the first failure. Other useful flags: `-w <group>`, `-s <package path|name>`, `-d <package>`, `-e <command>`.
- `scripts/build-all.js` builds TypeScript solution configs and pre-packs packages into `dist/`; useful flags: `-w <group>`, `-d <package>`, `--ts-config <file>`, `--no-install`, `--no-distws`.
- Main STH CLI source is `packages/sth/src/bin/hub.ts`; published/root bin points to `dist/sth/bin/hub.js`.
- Adapter-launched runner entrypoint is `packages/runner/src/bin/start-runner.ts`; executor selection is in `packages/runner/src/executor/select.ts`.
- Runtime wrapper packages (`runner-node`, `runner-python`) are protocol references for child process execution.

## Testing and generated files
- Package tests use AVA 8 and match `**/*.spec.ts`. AVA 8 imports test files as ESM, so `scripts/run-ava.js` stages TypeScript packages before the run: it copies the package into the `ava.typescript.rewritePaths` output directory (`.ava-<package>/`), compiles it with `typescript/bin/tsc`, symlinks sibling packages, rewrites escaping relative imports, and removes the staged tree after the run. Packages opt into this staging with `ava.typescript.compile: false` (all `packages/*` declare it).
- Run supported repo/package test commands with their default environment. Prefer a supported runner over raw test-process invocation.
- AVA package tests run through `scripts/run-ava.js` — the **sole supported** AVA/package-test entrypoint. All package `test`/`test:ava` scripts route through it. Default profile: `--max-old-space-size=2048`, JIT with WASM caps (8192 pages, 256 MB committed code/code space), `TS_NODE_TRANSPILE_ONLY=1`, concurrency 2, runner timeout 600000 ms. `SCRAMJET_TEST_PROFILE=fast` uses 16 AVA workers and an 8 MiB concurrent-mode budget; `SCRAMJET_TEST_PROFILE=phase-final` enables the unchanged strict 524288-byte guard and serial execution. Fast mode never enables concurrent GC measurements; an explicitly enabled guard remains serial. Opt in to jitless with `SCRAMJET_AVA_JITLESS=1`, or ts-node typechecking with `TS_NODE_TRANSPILE_ONLY=0`; package source TypeScript builds remain the correctness gate. Other overrides:
  - `SCRAMJET_AVA_FETCH=0` — adds `--no-experimental-fetch`
  - `SCRAMJET_AVA_WORKERS` — AVA concurrency (default 2)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scramjetorg/transform-hub](https://github.com/scramjetorg/transform-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
