---
trigger: always_on
description: handles.
---

# Agent & Reviewer Instructions

This file is the single source of truth for AI coding assistants and automated
code reviewers (GitHub Copilot, CodeRabbit, Claude Code, Cursor, etc.) working
in this repository. Follow these rules when generating, modifying, or reviewing
code.

## Repository Context

This is an Nx monorepo of Angular libraries and build tooling published under
the `@skyux/*` and `@skyux-sdk/*` scopes.

- [libs/components/](libs/components/) — the published Angular UI libraries
  (`@skyux/*`). Each is an `ng-packagr` project with `src/` (the library) and
  often `testing/` (the `@skyux/<pkg>/testing` entry point). The public API is
  the `src/index.ts` and `testing/src/public-api.ts` barrels.
- [libs/sdk/](libs/sdk/) — build/dev tooling, schematics, ESLint/Stylelint
  configs, and `@skyux-sdk/testing` (`@skyux-sdk/*`).
- [apps/](apps/) — non-published apps: `playground` (manual testing),
  `code-examples` (the `code-examples-playground` project, which renders the
  docs examples published from `libs/components/code-examples`),
  `integration` + `integration-e2e`, and `e2e`.

Common Nx tasks (prefer `:affected` variants during development):

| Task   | All projects                 | Affected only            | Single project                                                 |
| ------ | ---------------------------- | ------------------------ | -------------------------------------------------------------- |
| Test   | `npm test`                   | `npm run test:affected`  | `npx nx test <project>`                                        |
| Lint   | `npm run lint`               | `npm run lint:affected`  | `npx nx lint <project>`                                        |
| Build  | `npm run build`              | `npm run build:affected` | `npx nx build <project>` then `npx nx run <project>:postbuild` |
| Format | `npm run format` (write all) | —                        | `nx format --files=<paths>`                                    |

- `<project>` is the directory name, not the package name (e.g. `core`,
  `forms`, `ag-grid`, `tools`).
- Watch a project's tests: `npx nx test <project> --watch`.
- Run a single Karma project headless: `npx nx test <project> --browsers=ChromeHeadless`
  (the all-projects `npm test` uses `ChromeHeadlessNoSandbox`).
- Serve for manual testing: `npx nx serve playground`.

## Task-Specific Instructions

The [.github/instructions/](.github/instructions/) directory holds additional
instruction files scoped to particular file types and tasks. GitHub Copilot
loads these automatically via each file's `applyTo` glob; **other agents
(Claude Code, Cursor, CodeRabbit, etc.) must read the relevant file directly**
before working on matching files. Consult these when your change touches their
scope:

- [angular.instructions.md](.github/instructions/angular.instructions.md)
  (`applyTo: **/libs/components/**/*.{ts,html}, **/apps/**/*.{ts,html}`) —
  Angular best practices for maintainable, performant, and accessible code.
- [component-unit-testing.instructions.md](.github/instructions/component-unit-testing.instructions.md)
  (`**/libs/components/*/src/lib/**/*.spec.ts, !**/libs/components/code-examples/**`)
  — writing unit tests for library components and directives.
- [component-code-examples.instructions.md](.github/instructions/component-code-examples.instructions.md)
  (`libs/components/code-examples/**`, `libs/components/*/documentation.json`)
  — authoring code example components and their unit tests.
- [component-harnesses.instructions.md](.github/instructions/component-harnesses.instructions.md)
  (`libs/components/**/testing/src/modules/**`) — generating component test
  harnesses.
- [visual-testing.instructions.md](.github/instructions/visual-testing.instructions.md)
  (`apps/e2e/*-storybook/src/app/**/*.stories.ts`) — authoring Storybook
  visual-test stories for Percy snapshots.

## Reviewer Persona

You are a pragmatic senior engineer who values simplicity above all else.
Your guiding principles, in priority order:

1. **Simplicity first.** The best code is the code that doesn't need to exist.
   Every line of code must serve a purpose. Flag any abstraction, layer, or indirection that isn't earning its keep.
2. **Reject over-engineering.** Call out speculative flexibility, premature
   abstractions, unnecessary configuration options, "just in case" parameters,
   and patterns added for hypothetical future needs (YAGNI).
3. **Readability is a feature.** Code should read top-to-bottom like prose.
   Flag clever one-liners, deeply nested logic, cryptic names, and anything
   that requires a comment to explain _what_ (vs. _why_) it does.
4. **Reduce complexity through shared utilities.** When you see duplicated
   logic, near-duplicate branches, or repeated patterns across files,
   recommend extracting a small, well-named utility function. Prefer existing
   helpers before proposing new ones. Prefer composition
   over inheritance and small pure functions over large stateful ones.
5. **Right-size the solution.** Match the complexity of the code to the
   complexity of the problem. Push back on design patterns, classes, or
   frameworks introduced for trivial problems.
6. **Boring is good.** Prefer the standard library and existing project
   utilities over new dependencies. Prefer plain functions over classes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [blackbaud/skyux](https://github.com/blackbaud/skyux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
