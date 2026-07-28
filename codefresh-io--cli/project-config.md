---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

`codefresh` — the Codefresh command-line tool. A Node.js CLI built on `yargs` that talks to the Codefresh API through an OpenAPI-driven SDK (`codefresh-sdk`) and also wraps several external binaries (Stevedore, Venona, gitops controllers) on the user's machine.

Node `^24.0.0` is required (see [.nvmrc](.nvmrc)). Package manager is `yarn@1.22` with `--frozen-lockfile` in CI.

## Common commands

```bash
yarn install                                # install deps (use --frozen-lockfile in CI)
yarn test                                   # jest, runs every *.spec.js with --coverage
yarn test path/to/file.spec.js              # run a single spec file
yarn test -t "pattern"                      # run tests whose name matches pattern
yarn eslint                                 # lint + autofix lib/logic/**
yarn e2e                                    # bash-driven scenarios in e2e/scenarios/*.sh (needs an authenticated CLI)
yarn generate-completion                    # rebuilds lib/interface/cli/completion/tree.js — required by Dockerfile
yarn build-local-docs                       # generate Hugo docs into ./temp
yarn serve-docs                             # serve them locally; ALLOW_BETA_COMMANDS=true also includes beta cmds
yarn pkg                                    # build native binaries with @yao-pkg/pkg (alpine/macos/linux/win, x64+arm64)
node lib/interface/cli/codefresh <cmd>      # run the CLI directly from a clone (skipping the global `codefresh` install)
```

Jest setup is configured in [package.json](package.json) (`jest.setupFiles` → [test-setup.js](test-setup.js)). The setup mocks `Output`, `whoami`, and `request-promise`, and exposes `configureSdk()` / `verifyResponsesReturned()` as globals used throughout `*.sdk.spec.js`.

## Architecture

### Entry point and command discovery

`lib/interface/cli/codefresh` (the `bin` entry) delegates to [commad-line-interface.js](lib/interface/cli/commad-line-interface.js) (note the spelling). On startup it:

1. Recursively reads every `*.cmd.js` file under [lib/interface/cli/commands/](lib/interface/cli/commands/).
2. Loads SDK config (`codefresh-sdk` + bundled [openapi.json](openapi.json) + cfconfig auth) — unless the invocation is `config use-context`, which is the only context-less path.
3. Registers only commands where `isRoot()` is true. Subcommands attach to their parent via the `parent:` option in their constructor (see below).

Beta commands are filtered at this top level by `context.isBetaFeatEnabled()`. The on-prem flag is filtered per-subcommand inside `Command._createBuilderFunction`.

### The `Command` class

[lib/interface/cli/Command.js](lib/interface/cli/Command.js) wraps yargs. Conventions every command file follows:

- Each file exports a single `new Command({...})` instance (file name `*.cmd.js`).
- Root commands set `root: true`; subcommands set `parent: require('../some/parent.cmd')` — that call also auto-registers the child via `setParentCommand`.
- `requiresAuthentication` defaults to `true` for roots; subcommands inherit unless they override. Auth is enforced in [helpers/general.js#wrapHandler](lib/interface/cli/helpers/general.js) which also implements `--watch` (loop + `draftlog`) and the consecutive-error tolerance for watch mode.
- `betaCommand: true` / `onPremCommand: true` gate the command behind context feature flags.
- `webDocs: { category, title, weight }` feeds the Hugo docs generator ([docs/index.js](docs/index.js)) via `prepareDocs()`.
- Handlers receive parsed `argv`; values can also be supplied through `CF_ARG_*` env vars (set in `commad-line-interface.js` via `yargs.env('CF_ARG_')`).

When adding a new command, follow the surrounding directory's pattern — most resources have a `get.cmd.js` / `create.cmd.js` / `delete.cmd.js` triple, and bash completion is generated from sibling `*.completion.js` files.

### SDK layer

[lib/logic/sdk/index.js](lib/logic/sdk/index.js) exports a single shared `Codefresh` SDK instance. It is configured once at startup from `openapi.json`; downstream code does `const { sdk } = require('../../logic')` and calls methods like `sdk.pipelines.get({...})` whose names come straight from the OpenAPI spec.

To add a new API endpoint there is usually no manual SDK wiring — regenerate/bump `openapi.json` and the method appears on `sdk`.

### Output and entities

`Output.print(data)` ([lib/output/Output.js](lib/output/Output.js)) dispatches by `--output` flag to a renderer in [lib/output/types/](lib/output/types/) (`json`, `yaml`, `table` (default + wide), `id`, `name`, `jsonArray`, `yamlArray`). When `--watch` is set, output goes through `draftlog` to overwrite in place.

Domain objects under [lib/logic/entities/](lib/logic/entities/) extend [Entity.js](lib/logic/entities/Entity.js). Each entity declares `defaultColumns` / `wideColumns`; renderers call `toDefault()` / `toWide()` / `toJson()` / `toYaml()` / `toName()` / `toId()`. Table column styling (colors, status badges) lives in matching files under [lib/output/formatters/](lib/output/formatters/). `fromResponse` is the constructor convention used by command handlers.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [codefresh-io/cli](https://github.com/codefresh-io/cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
