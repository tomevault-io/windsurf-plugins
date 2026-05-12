---
trigger: always_on
description: > botscript's contribution guide, written for bots.
---

# AGENTS.md

> botscript's contribution guide, written for bots.

This project is intentionally built so that the *primary* contributors are LLMs
(Claude, Codex, Gemini, DeepSeek, future ones we haven't met yet). Humans
review the philosophy and the diff; bots write the code, the tests, and the
docs. This document is the contract that makes that work.

If you are a model reading this in the middle of a task: **stop, read, then
proceed**. Most of what would otherwise be guesswork is fixed below.

---

## Architecture in 90 seconds

```
botscript/
├── packages/
│   ├── runtime/        Pure JS/TS. Result, Option, $match, $enter, $assert.
│   ├── compiler/       String-in / TypeScript-out transformer. Pass pipeline.
│   ├── cli/            `botscript build`, `botscript primer`. Thin wrapper.
│   ├── vite-plugin/    Vite integration. Calls compiler, then esbuild for JSX.
│   └── babel-plugin/   Babel integration. parserOverride.
├── examples/
│   ├── node-app/       CLI — shapes/area/match end-to-end.
│   └── react-app/      Vite + React — todo list, tests in .bs.
├── MANIFESTO.md        Why this project exists. Read first.
└── STDLIB.bs           Canonical example of every language feature, exactly once.
```

The compiler is a sequence of string-to-string passes (`packages/compiler/src/passes/`),
each of which targets a single syntactic form. There is **no AST** — every pass
uses bracket-aware string scanning helpers in `lex.ts`. This is deliberate:
the entire compiler fits in one head, and a bot adding a new feature touches
exactly one new file plus the pipeline list in `transform.ts`.

## The contribution rules

These are non-negotiable. CI enforces them; humans don't.

### 1. Every change ships with a test.

If you add a transform, add a test for the form it rewrites *and* a test for
a form that should NOT be rewritten (the no-op case). If you add a runtime
helper, add a test for the happy path *and* the failure mode. Tests live in
`packages/<pkg>/tests/`.

### 2. The primer is the contract.

`packages/compiler/src/primer.ts` is the single source of truth for what the
language is. If your change adds new syntax, the primer must also change in
the same PR. If your change does not add syntax, the primer must not change.
Drift here is the only thing in this codebase that compounds.

### 3. Anything you do must work end-to-end in `examples/`.

Adding a feature isn't done when tests pass — it's done when at least one
example demonstrates it. If your feature isn't useful enough for the canonical
node or react example, it's probably not useful enough to ship.

### 4. Versioned syntax is forever.

A `.bs` file pinned to `?bs 0.1` must compile identically a year from now.
New syntax goes behind a new version pin (`?bs 0.2`, `?bs 0.3`). The
`SUPPORTED_VERSIONS` array in `packages/compiler/src/passes/version.ts` is the
list every other component branches on. Never modify a shipped version's
behavior in place.

### 5. Don't add abstractions speculatively.

The compiler has six passes because that's what botscript currently does. It
does not have a pluggable pass registry, a configuration object, or a trace
mode. Add those when a real second use case appears, not before. (See
"Be small. Stay small." in the manifesto.)

### 6. Run `pnpm test` and `pnpm -r build` before claiming done.

In that order. If either fails, the change is not done. You may not skip
tests with `.skip` to land a PR; if a test is wrong, fix the test in the
same PR and explain why.

### 7. No emoji in code, comments, or commit messages.

The user has zero patience for it. Default to plain text everywhere.

### 8. Commits are atomic.

One commit, one feature/fix. The commit message starts with the package name
in scope: `compiler: add support for ?ext directive`. Body explains why,
not what. Don't reference issue numbers in the title.

## How to add a feature (the recipe)

The bot-optimized version of the contributor recipe. Steps 1-9 are mandatory;
skipping any of them is what causes "I added a feature, why doesn't anyone
know about it?" drift between the compiler, the docs, and the bots.

1. **Pick a target form.** What syntax are you adding? Write down the exact
   `.bs` snippet you want to support, and the TypeScript it should desugar to.
   Both must be in the PR description.
2. **Pick a version pin.** New syntax goes behind a new pin if (and only if)
   the change can break already-shipped files at the previous pin. A purely
   additive feature (new keyword, new block form) can land at the current
   `LATEST` pin. A behaviour change to existing forms requires bumping
   `SUPPORTED_VERSIONS` in `packages/compiler/src/passes/version.ts` and
   gating internally.
3. **Update `STDLIB.bs`.** Add one example of the new form. If you can't write
   one, the feature is probably not coherent yet.
4. **Update `primer.ts`.** Add the new form to the right section. Keep the
   primer under one screen — if the feature can't be described in three lines,
   reconsider its scope.
5. **Add a new pass.** Create `packages/compiler/src/passes/<name>.ts`.
   Export a single function `(src: string, version: VersionInfo) => string`
   (the second arg is optional — accept it if the pass branches on version).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marcelofarias/botscript](https://github.com/marcelofarias/botscript) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
