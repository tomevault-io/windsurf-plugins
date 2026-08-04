---
trigger: always_on
description: This is durable context. The current task / phase status lives in
---

# Notes for Claude / coding agents working on GaleforceCSS

This is durable context. The current task / phase status lives in
[`todo.md`](./todo.md); the long-form plan lives there too.

## What this project is

A Rust-powered, **Tailwind CSS v3-compatible** compiler with a Vite
plugin. Compatibility is pinned to `tailwindcss@3.4.19` — that exact
release is the conformance oracle. We are not Tailwind Labs; we are
re-implementing the no-plugin v3 surface in Rust for speed and stability.

## Porting philosophy

**Treat this as a port, not a clean-room reimplementation.** When you
need to make a behavioral decision — what does this variant emit, what
does this directive expand to, how does this util look up a theme key —
the answer comes from reading the JS source at the path below, not from
intuition. Mirror Tailwind's *observable output* exactly (modulo the
PostCSS normalizer); the conformance harness will catch you if you
guess.

Where the JS implementation has tests that capture a behavior corner
(e.g. `tests/apply.test.js`, `tests/variants.test.js`), use those as
the basis for our Rust tests — same inputs, same expected outputs. A
test that ports cleanly is a test we can trust.

That said, **internal implementation is free to differ** wherever it
doesn't change observable output:

- Content scanning, file I/O, and the candidate cache can be parallelised
  or rewritten with different data structures — Tailwind is single-
  threaded JS; we don't have to be.
- Bracket-aware tokenizers, selector escape, sort-key construction, etc.
  can be byte-tables or hand-rolled state machines instead of regex.
- The CSS AST shape, the rule-emit format, and crate boundaries are
  ours to choose.

The rule of thumb: **mirror the source for behavior, mirror the tests
for confidence, but write idiomatic Rust for everything else.**

## Authoritative references

When a question is ambiguous, do not guess. Read the source. There are
two pinned copies of `tailwindcss@3.4.19` in this repo and they serve
different purposes:

- **`vendor/tailwindcss-v3/`** is a git submodule pinned at the v3.4.19
  commit (`f38be227df384504a170409c2131ca5ca8bfe025`). This is the
  authoritative source path — it includes `tests/` (28k lines of upstream
  test cases we can mine for fixture inputs and expected outputs).
  Important paths:
  - `src/corePlugins.js` — every utility/variant definition
  - `src/util/escapeClassName.js`, `src/util/escapeCommas.js` — selector escape
  - `src/util/defaultTheme.js`, `stubs/config.full.js` — default theme values
  - `src/css/preflight.css` — preflight reset
  - `src/lib/setupContextUtils.js` — candidate parsing, variant logic
  - `src/lib/expandTailwindAtRules.js` — `@tailwind` / `@layer` / `@apply`
  - `src/lib/expandApplyAtRules.js` — `@apply` resolution
  - `src/lib/evaluateTailwindFunctions.js` — `theme()` / `screen()` calls
  - `src/lib/substituteScreenAtRules.js` — `@screen md` rewriting
  - `tests/apply.test.js`, `tests/variants.test.js`, `tests/dark-mode.test.js`,
    `tests/arbitrary-values.test.js`, `tests/escapeClassName.test.js`, etc.
    — port edge cases from these into our fixtures.

  Bootstrap with `git submodule update --init --recursive` after a
  fresh clone. `pnpm oracle:version` verifies the submodule + npm
  versions match.

- **Executable oracle**: `tailwindcss@3.4.19` is also installed as an
  npm dependency at `node_modules/.pnpm/tailwindcss@3.4.19_*/`. Same
  bytes as the submodule, but importable into Node code — that's how
  `@coralogix/galeforcecss-oracle` calls into Tailwind for conformance testing. Use
  this when you want to run `compileWithTailwind3` to see what Tailwind
  emits for some input.

  Use the **JS reference**, not your memory, when implementing or
  debugging an escape rule, parsing detail, or default theme value.

- **Conformance harness** (`packages/galeforcecss-conformance`) compares your
  output against the live oracle output. Semantic equivalence after the
  PostCSS-based normalizer — not byte equality. If you wonder "what
  would Tailwind emit here?", the answer is `pnpm exec tsx -e "…"` with
  `compileWithTailwind3`.

- **`todo.md`** is the long-form spec and Phase plan.

## Repo layout

```
crates/                       # Rust workspace
  galeforce-core/                # shared types: errors, diagnostics, options, results
  galeforce-scanner/             # file walking + candidate tokenizer + incremental cache
  galeforce-parser/              # candidate parser (variants, important, prefix, modifier)
  galeforce-theme/               # theme resolution (stub)
  galeforce-css/                 # CSS AST + selector escaping + emitter
  galeforce-sort/                # rule ordering (stub)
  galeforce-compiler/            # parser -> utility lookup -> Rule, with diagnostics
  galeforce-cli/                 # `galeforcecss compile-json` (and others, stubbed)
  galeforce-node/                # napi-rs bindings (stub; not wired yet)

packages/                     # pnpm workspace
  galeforcecss/                  # public Node API (stub)
  vite-plugin-galeforcecss/      # Vite plugin (stub)
  galeforcecss-oracle/           # wraps tailwindcss@3.4.19 for conformance
  galeforcecss-conformance/      # fixture runner; spawns target/release/galeforcecss


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coralogix/galeforce-css](https://github.com/coralogix/galeforce-css) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
