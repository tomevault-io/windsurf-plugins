---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

`plain-forge` is an npm package with two distinct halves:

1. **A tiny installer CLI** (`bin/cli.mjs`) — `npx plain-forge install|update|uninstall`.
2. **The actual product** under `forge/` — a library of AI-agent *skills* and *rules* that teach an agent to author and maintain `***plain` specification files. The CLI copies `forge/` verbatim into an agent directory; there is **no build step** and no generated/committed output.

So a change here is almost always one of: (a) editing the installer CLI, or (b) editing the instructional content under `forge/skills/` and `forge/rules/`. These need different mindsets — see "Editing `forge/` content" below.

`plain-forge` only *authors* `.plain` specs. Rendering specs into code is done by a **separate** tool, the `codeplain` CLI (codeplain.ai), which this repo does not contain.

## Commands

```bash
npm test                                                   # full suite: node --test test/*.test.mjs
node --test --test-name-pattern="<regex>" test/*.test.mjs  # run a single test by name
node --test test/cli.test.mjs                              # run a single test file
```

- Test runner is **Node's built-in `node:test`** (`node:assert/strict`) — not jest/vitest, no test framework dependency. Requires Node ≥18.
- `test/cli.test.mjs` mixes unit tests (importing named exports from `bin/cli.mjs`) with black-box integration tests that `spawnSync` the real CLI into isolated temp `HOME`/cwd dirs.
- Releases are cut by publishing a GitHub Release tagged `vX.Y.Z`, which publishes to npm via OIDC — see [RELEASING.md](RELEASING.md). The version lives in the tag, so `package.json` on `main` is intentionally stale.
- **There is no working build or lint step.** `package.json` declares `build`/`clean` scripts (`tsx bin/forge-build.ts`) and `tsconfig.json` references `bin/**/*.ts` + `runtimes/**/*.ts`, but **those files do not exist** — `npm run build`/`npm run clean` error out. The TS toolchain (tsx/typescript) is vestigial; the CLI is plain `.mjs` run directly by Node. Don't try to build.

## The installer CLI (`bin/cli.mjs`)

A single self-contained ESM file with **zero runtime dependencies**; it exports its internals so the test suite can import them without running `main()` (guarded by `isInvokedDirectly()`, which realpath-compares `argv[1]` to `__filename` — needed because the global bin is a symlink). Key model:

- `AGENTS` maps agent name → content dir: `claude→.claude`,
  `codex|copilot|universal→.agents`, `forgecode→.forge`, and `opencode→.opencode`.
  `SCOPES`: `project` (cwd) / `global` (`$HOME`). Global ForgeCode and OpenCode paths have explicit
  exceptions in `resolveBaseDir`. `CONTENT_DIRS = [skills, rules, docs]` (missing source dirs are
  silently skipped).
- **install** writes `forge/{skills,rules,docs}` into `<agentDir>/`, recording every written file in `<agentDir>/.plain-forge/manifest.json`. It **refuses** (exit 1) if a manifest or a "forge signature" already exists — install never overwrites in place; you use `update` for that.
- **update** auto-detects every install across both scopes × all agents, re-copies the fresh tree, and **prunes** files that were in the old manifest but no longer ship (confirmed individually unless `--yes`). Only manifest-recorded files are ever prune candidates, so the user's own/third-party files are never touched.
- **uninstall** deletes exactly `manifest.files` then the manifest; refuses (exit 1) on a manifest-less install rather than guessing which files are its own.
- Legacy (manifest-less) installs are recognized only when **all** of `FORGE_SIGNATURE_SKILLS` (`forge-plain`, `add-feature`, `debug-specs`, `load-plain-reference`) are present, then refreshed and given a manifest going forward.

When changing install/update/uninstall behavior, update `test/cli.test.mjs` accordingly — it is the spec for this CLI.

## Editing `forge/` content (skills + rules)

This is the heart of the product. Everything under `forge/` is **instructional text consumed by an AI agent at author-time**, not code that executes. Four consequences:

- **`forge/rules/` is the sole source of truth for writing `.plain`.** Every syntax rule,
  section-ownership rule, constraint, and canonical `.plain` example belongs in the applicable rule
  file. Do not duplicate authoring guidance in skills or skill references; duplication drifts and
  creates contradictory instructions.
- **`load-plain-reference` is a router, not a language guide.** Its `SKILL.md` stays concise and maps
  the current task to only the applicable files under `forge/rules/`. It may route to focused
  operational references for project layout or rendering/testing, but those references must not
  restate `.plain` authoring rules. Integration rules load only for integration work. Claude skips
  rereading rules already supplied natively; other agents read only rules not already in context.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [plainlang/plain-forge](https://github.com/plainlang/plain-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
