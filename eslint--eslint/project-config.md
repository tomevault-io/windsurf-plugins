---
trigger: always_on
description: This file provides guidance to AI agents when working with code in this repository.
---

# AI Agent Instructions

This file provides guidance to AI agents when working with code in this repository.

## AI Disclosure Requirement

ESLint's [AI Usage Policy](docs/src/contribute/ai-policy.md) requires that AI-assisted contributions be disclosed. Whenever you (an AI) create content on GitHub for this project, you **must** include a disclosure, using the name of the model actually producing the content (for example, `Claude Opus 5`):

- **Issues** — add a bold sentence to the top of body:

    **This issue was created with AI (Claude Opus 5).**

- **Pull requests** — add a bold sentence to the top of the description:

    **This pull request was created with AI (Claude Opus 5).**

- **Comments, review comments, and review summaries** — add a final paragraph in bold:

    **Created by AI (Claude Opus 5).**

This applies to every AI-authored submission without exception, including follow-up comments on an existing thread. Note two related policy points: AI-generated PRs are only considered for issues labeled `accepted`, and maintainer feedback is expected to be answered by a human rather than fed back into an AI.

## Commands

```bash
npm test                 # full suite: rule-file checks, mocha + coverage thresholds, fuzzer, license check
npm run test:cli tests/lib/rules/no-undef.js   # run a single test file (alias for local mocha)
npm run lint             # lint everything (JS, docs JS, docs Markdown, rule types, config files)
npm run lint:fix
npm run fmt              # prettier --write . (prettier is the formatter; eslint does not handle style here)
npm run test:types       # tsc against tests/lib/types
npm run test:browser     # cypress against the webpack bundle
npm run test:performance
```

Useful details:

- Coverage gates are enforced in `npm test` (99% statements/functions/lines, 98% branches). A change that lowers coverage below those thresholds fails the build even if all tests pass.
- Mocha's default timeout is 10000ms; override with `ESLINT_MOCHA_TIMEOUT=20000 npm test`.
- `npm test` runs mocha with `--forbid-only`, so `only: true` / `RuleTester.only(...)` must be removed before pushing.
- Task definitions live in `Makefile.js` (shelljs-based), not a Makefile. `npm run lint`, `npm test`, etc. are thin wrappers around `node Makefile.js <target>`.
- The docs website is a separate workspace with its own scripts: `cd docs && npm start` serves it locally.
- A `lint-staged` pre-commit hook regenerates derived files. Editing `lib/rules/*.js` regenerates `packages/js/src/configs/*.js` and `lib/types/rules.d.ts`; editing `docs/src/rules/*.md` regenerates `docs/src/_data/further_reading_links.json`. Don't hand-edit those generated files.

## Architecture

Beyond `lib/` (the source) and `tests/` (which mirrors it), the top-level directories are `bin/` (CLI entry point), `conf/` (configuration data), `docs/` (the documentation website), `messages/` (verbose text for certain runtime errors), `packages/` (separately published packages), `templates/` (templates for generated files), and `tools/` (build, release, and check scripts).

The layering is strict, and each layer is forbidden from doing what the layer below it does. Respect these boundaries — tests and reviews enforce them.

- `bin/eslint.js` → `lib/cli.js` → `lib/eslint/eslint.js` → `lib/linter/linter.js` → `lib/rules/*.js`
- **`lib/cli.js`** is the only place that reads argv, writes to the console, and sets exit codes. It may not call `process.exit()` directly.
- **`lib/eslint/`** (`ESLint` class) owns all file system access: file/glob resolution, config loading, plugin and formatter loading. It must not print anything or use a formatter itself. `lib/eslint/worker.js` supports multithreaded linting.
- **`lib/linter/`** (`Linter` class) is pure and synchronous: no file I/O, no console, no Node-specific APIs, no async. `verify()` parses text, traverses the AST, and emits node-type events (plus `:exit` events and code path analysis events from `lib/linter/code-path-analysis/`) that rules subscribe to.
- **`lib/rules/`** rules are the most constrained layer: inspect the AST, report problems. Same prohibitions as `Linter`.
- **`lib/config/`** implements flat config: `config-loader.js` finds and loads `eslint.config.js`, `flat-config-array.js` and `flat-config-schema.js` normalize and validate it, `default-config.js` supplies base values.
- **`lib/languages/js/`** is the JavaScript language implementation, including `SourceCode`. ESLint's language plugin abstraction means JS is one language among potential others, so language-specific logic belongs here rather than in `Linter`.
- **`lib/rule-tester/`** is `RuleTester`, a wrapper over Mocha-style globals used by essentially every rule test.
- **`lib/shared/`** is cross-cutting utilities (`flags.js` for feature flags, `traverser.js`, severity/naming/serialization helpers).
- **`lib/services/`** holds parser, processor, suppressions, and warning services used by `ESLint`.
- **`packages/js`** (`@eslint/js`) publishes the `recommended` and `all` configs, generated from rule metadata. **`packages/eslint-config-eslint`** is the config this repo lints itself with.

## Rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eslint/eslint](https://github.com/eslint/eslint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
