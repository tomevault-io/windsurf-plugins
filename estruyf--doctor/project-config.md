---
trigger: always_on
description: Guidance for AI coding agents (Claude Code, Copilot, Cursor, Codex, …) working in this repository.
---

# AGENTS.md

Guidance for AI coding agents (Claude Code, Copilot, Cursor, Codex, …) working in this repository.
Humans are welcome to read it too — it is the fastest description of how this project hangs together.

## What this is

`doctor` (`@estruyf/doctor`) is a CLI that publishes a folder of Markdown files as SharePoint pages —
a static site generator that outputs SharePoint pages instead of HTML files. It talks to SharePoint
through the [CLI for Microsoft 365](https://pnp.github.io/cli-microsoft365/) (`@pnp/cli-microsoft365`),
with a few direct REST calls where that CLI has no command.

Docs live at [getdoctor.io](https://getdoctor.io) and are built with [Astro Starlight](https://starlight.astro.build/)
from [docs/](docs/) — **not** Hugo (the site was migrated; ignore leftover Hugo-style paths referenced elsewhere).

Node.js >= 22.13.0. ESM (`"type": "module"`) — **relative imports must carry the `.js` extension**, even
from `.ts` sources. TypeScript 5, `module: nodenext`, `strictNullChecks` and `noUnusedLocals` are on.

## Commands

```bash
npm run build       # clean + tsc + tsc-alias   (tsc-alias rewrites the @-aliases in dist)
npm test            # build, then: node --test tests/**/*.test.mjs
npm run watch       # tsc -w + tsc-alias -w
npm run watch:debug # same, but patches bin/doctor to `node --inspect` + DEBUG=true
npm run docs        # astro dev — serve the Astro Starlight docs site from docs/
npm run docs:build  # npm install + astro build in docs/ (what the site deploy runs)
```

Run a single test file (tests import from `dist/`, so build first):

```bash
npm run build && node --test tests/locale.test.mjs
```

Tests are plain `node:test` `.mjs` files in [tests/](tests/) that import compiled helpers from `../dist/...`.
There is no linter and no test framework beyond the Node test runner — do not introduce one without being asked.

`npm link` makes the local build available as the global `doctor` command. The sample content repo is
[estruyf/doctor-sample](https://github.com/estruyf/doctor-sample) (what CI publishes against).

## Architecture

### Request flow

`bin/doctor` → [src/cli.ts](src/cli.ts) → [src/main.ts](src/main.ts) → a command class in [src/commands/](src/commands/).

Commands are the `Command` enum in [src/commands/Command.ts](src/commands/Command.ts):
`init`, `publish`, `workflow`, `status`, `version`, `setup`, `cleanup` (the last two wire up shell autocomplete).

1. [OptionsHelper](src/helpers/OptionsHelper.ts) reads `doctor.json` from `process.cwd()`, merges CLI
   arguments over it (`arg` package), then prompts via `inquirer` for anything still missing. Everything
   ends up in one flat `CommandArguments` object threaded through the whole run.
2. `Commands.start()` calls `resetRuntimeState()` and initializes `Logger`/`OutputHelper`/`CliCommand`/`PartialsHelper`,
   then dispatches on `options.task`.
3. Long-running commands (`publish`, `status`) render as a `listr2` task list. Each publish step is a task
   with an `enabled:` predicate driven by the options — that list in [publish.ts](src/commands/publish.ts) is
   the clearest description of the pipeline: clean → multilingual config → load state → collect markdown →
   precheck → pages → localized pages → remove deleted → navigation → site design → post cleanup → save state.

### Adding an option — three places, always

An option that is missing from any of these is silently ignored at runtime:

1. `OptionsHelper.getArgs()` — the `arg` definition (`--flag`).
2. `OptionsHelper.parseArguments()` — the merge, in the order **argument → `doctor.json` → default**.
   **Arguments always win over `doctor.json`.**
3. The matching interface in [src/models/CommandArguments.ts](src/models/CommandArguments.ts)
   (`RuntimeOptions`, `AuthOptions`, `PublishOptions`, `ContentOptions`, `NavigationOptions`,
   `SiteOptions`, `TaskToggleOptions`).

Nested `doctor.json` settings (`markdown.*`, `partials.*`, `multilingual.*`, `menu`, `siteDesign`) are
flattened here — e.g. `markdown.shortcodesFolder` becomes the top-level `shortcodesFolder`.

Then: update `schema/<version>.json` (the highest-numbered file in [schema/](schema/) is the current one)
and the docs page — see [Definition of done](#definition-of-done).

### Static classes with mutable state

Nearly every helper is a class of `static` methods holding module-level state (page cache, publish state,
counters, partials cache, rendered-diagram cache). This only works because `Commands.resetRuntimeState()`
clears them all at the start of a run — **any helper that keeps state must expose `reset()` and be added to
that list in [main.ts](src/main.ts)**, otherwise tests (and repeat runs in the same process) leak state
between cases. Tests call the compiled helpers directly, so a forgotten `reset()` shows up as an
order-dependent test failure.

### Output: one stdout writer

[OutputHelper](src/helpers/OutputHelper.ts) is the single place that writes to stdout. With `--output json`
the human-readable output has to disappear completely, and the run ends with exactly one JSON document
written by `flush()`. **Never `console.log` from a helper** — go through `OutputHelper`. Debug output goes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [estruyf/doctor](https://github.com/estruyf/doctor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
