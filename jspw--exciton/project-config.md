---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

exciton is a CLI (`exciton` / `xc`) that runs Claude Code with an agentic workflow framework (currently only `superpowers`) dialled to a chosen profile — full, or `--no-hooks` (skills stay callable, nothing auto-fires) — for a single session, without writing anything under `~/.claude`. A framework must be **added** (`exciton add`) before it will run; first invocation with no config runs an onboarding walkthrough. It does this using two documented Claude Code primitives: `--settings '{"enabledPlugins":{...}}'` to disable plugins for the session, and `--plugin-dir <dir>` to add one in. See [MECHANISM.md](MECHANISM.md) for the full mechanism and verification evidence, [PRODUCT.md](PRODUCT.md) for scope, and [QA.md](QA.md) for the reasoning behind specific design decisions.

## Commands

```sh
npm run build              # tsc: src/ -> dist/
npm test                   # unit tests: node --test --test-concurrency=2 test/*.test.ts
npm run test:integration   # integration tests: shells out to a real `claude` binary; needs claude installed
```

Run a single unit test file directly, e.g.:

```sh
node --test --test-concurrency=2 test/resolve.test.ts
```

There is no separate lint command. `tsc` (via `npm run build`) is the type check; `strict` mode is on.

Source files are plain `.ts` run directly by Node (no build step needed for tests) — this repo relies on Node ≥22.18's unflagged TypeScript stripping, which is why that version is the floor in `package.json` `engines` and why CI matrixes `22.18.0` and `24.x`. `npm test` runs `test/*.test.ts` directly, not compiled output.

## Architecture

The whole CLI is ~1,800 lines across `src/`. Read the files, not a summary — but here's the shape of how a run flows through them, since that requires connecting several files:

1. **`cli.ts`** — entry point. `parseArgs` splits argv on `--` (everything after is forwarded verbatim to `claude`); before it, one positional arg is the framework name/path/spec, `--no-hooks` picks the profile, subcommands (`add`, `remove`, `update`, `list`, `clean`, `help`, `version`) short-circuit before framework resolution. `run()` orchestrates the rest of the pipeline below and is the place to look first when tracing behavior. It also fires onboarding on first contact and enforces `assertAdded` — a framework absent from the registry does not run.
2. **`registry.ts`** — `~/.exciton/config.json`: which frameworks have been added and which copy each runs from (`installed` = Claude's, `own` = exciton's clone). Split into two I/O functions taking an injectable path and six pure ones. `onboardedAt` distinguishes "never onboarded" from "onboarded and chose nothing" — without it, opting out would re-trigger the walkthrough forever.
3. **`resolve.ts`** (`resolvePlugin`) — turns a bare name, a full plugin id (`name@marketplace` — the marketplace half is ignored, since exciton has no version syntax), or a path spec into a `Resolved` (dir, version, sha, origin). Tries, in order: path spec → already-installed plugin (`installed.ts`) → a marketplace entry (`marketplace.ts`) cloned via `fetch.ts` into exciton's own cache. **`{ ownCopy: true }` skips the installed lookup** — without it `source: 'own'` would silently resolve to Claude's copy and the choice would be decorative. `fetch.ts` resolves the newest *release tag* (`git ls-remote --tags --refs`, numeric compare, pre-releases excluded) and clones it in one `git clone --depth 1 --branch <tag>`.
4. **`frameworks.ts`** — the `FRAMEWORKS` set (currently just `superpowers`) is the single source of truth for what exciton is allowed to manage. `cli.ts` uses `assertManaged`/`assertSingleFramework` to refuse anything not in this set and refuse naming two frameworks at once — frameworks are mutually exclusive by design (they compete to define how a session is conducted), while ordinary plugins are left completely untouched.
5. **`settings.ts`** (`collectPluginIds`) — reads `enabledPlugins` across every settings scope (user, project, project-local, enterprise-managed) to find every *managed-framework* id currently enabled anywhere, even ones not named on the command line — those must still be suppressed, or a second framework would keep silently governing the session. `buildDisablePayload` turns that into the `--settings` JSON, touching only the `enabledPlugins` key (any other key would outrank project/local settings).
6. **`stage.ts`** (`stagePlugin`) — for the `full` profile, points `--plugin-dir` straight at the resolved source (zero copy). For `nohooks`, atomically builds a cached copy under `~/.exciton/staged/` with the `hooks/` directory filtered out, keyed by name+version+sha so it's built once. Hooks are discovered by convention in Claude Code, so removing the directory is enough to make nothing auto-fire while skills remain callable.
7. **`launch.ts`** — spawns `claude` with inherited stdio (`spawnSync`, not `execve` — Node has none) and forwards its exit code; builds argv as `--settings <payload>? --plugin-dir <dir>... <forwarded args>`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jspw/exciton](https://github.com/jspw/exciton) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
