---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

A Svelte 5 port of [Astryx](https://astryx.atmeta.com/), Meta's open source design system. pnpm
monorepo: `packages/core` (components), `packages/cli`, `packages/themes/*`, `docs`.

## The parity rule

**If it's not in Astryx, it's not here.** Invented props, extra variants, nicer defaults and
hand-drawn demo content are _defects_, not improvements — that includes the demo routes, which must
show upstream's documented API. Upstream bugs are documented in `TODO.md` under "Known debts" rather
than replicated.

Upstream's source is cloned at **`reference/astryx-upstream/`** — gitignored, present locally, and
the thing to read. Read it _before_ porting, not after: source, `.doc.mjs`, tests, storybook, and
the compiled `dist/` in `node_modules/@astryxdesign/core`.

**After cloning it, rename its `CLAUDE.md` to `UPSTREAM-CLAUDE.md`.** Reading any file in that tree
otherwise loads Meta's instructions for _their_ repo into agent context, where they read as
instructions for this one — they describe a React codebase, a different test runner and a CLI this
repo does not have. The content is kept under the new name, and its StyleX capability table is
worth consulting; it just must not auto-load.

`TODO.md` is the live status and backlog (what's next, the batch history, known debts). Read the
relevant section before starting work; update it when work lands. **`PORTED.md` is its companion**
— the per-component implementation notes (what each unit does, the translations it needed, its
oracle and test posture). Status and open decisions go in `TODO.md`; how a component was built goes
in `PORTED.md`.

## Subagents

Five agents in `.claude/agents/` own distinct axes. Use them — they encode this port's failure modes.

- `astryx-parity` — props, styles, elements, exports vs upstream. Run **before** porting (to extract
  the spec) and **after** (to catch drift).
- `astryx-idiom` — whether the React→Svelte translation is _correct_: contexts storing values instead
  of getters, `$derived` caching through a server render, un-`untrack`ed attachments. Exactly the
  class of bug `astryx-parity` is told to ignore.
- `astryx-oracle` — wires a `.stylex.ts` module into the class oracle and diagnoses mismatches.
- `astryx-test-parity` — ports an upstream `.test.tsx` suite case-for-case.
- `astryx-surface` — repo-wide published-API sweep. Run after a batch lands.

## Commands

```sh
pnpm -r build     # must run before check — theme-neutral typechecks against core's built dist/,
                  #   and the docs generator reads props types out of that same dist/
pnpm -r check     # svelte-check + tsc
pnpm -r lint      # prettier --check && eslint
pnpm -r test      # vitest + all three fidelity oracles
pnpm -F @astryx-svelte/core test:unit --run      # unit tests only
pnpm -F @astryx-svelte/core test:unit --run src/tests/foo.svelte.test.ts   # one file
#   NOTE: no `--`. Under pnpm 10 the `--` is passed through, so vitest sees
#   `"--" "--run" "<path>"`, ignores both the flag and the filter, and starts a
#   full run in *watch mode* that never exits. It looks exactly like a hang.
#   `--project=client` / `--project=server` narrows to one vitest project.
pnpm -F @astryx-svelte/core test:client   # the client project, chunked — see below
#   The client project cannot be run in one process: it dies partway through with
#   `wrapDynamicImport` of undefined (Vite's module runner, not an assertion) and
#   reports every later file as failed. Measured on both Windows and Ubuntu CI, at a
#   different file each time. `scripts/run-client-tests.mjs` runs it in batches of 20
#   (`CLIENT_CHUNK_SIZE` overrides) and reconciles files-run against files-on-disk, so
#   a chunk that collected nothing fails the run instead of shrinking the total. This
#   is what `core`'s `test` script and CI both use; a bare `--project=client` over all
#   162 files is not a measurement.
pnpm dev          # core's demo routes;  pnpm dev:docs for the docs site
pnpm -F docs generate   # regenerate the docs content registries (runs automatically on dev/build)
```

Never install with `--prod` or prune devDependencies: both oracles **and the docs content
pipeline** read the upstream `@astryxdesign/*` packages, which are devDependencies.

## The docs site

`TODO.md`'s Phase 5 is the current goal. Two things about `docs/` are easy to get wrong:

- **It compiles core's StyleX itself.** `dist/` ships `.stylex.js` _uncompiled_ — `svelte-package`
  transpiles TypeScript and does not run StyleX — so `docs/vite.config.ts` runs the same
  `@stylexjs/unplugin` options as `packages/core`, plus `optimizeDeps.exclude` and `ssr.noExternal`
  for `@astryx-svelte/core`. Without those last two, Vite's pre-bundler and the SSR externaliser
  route the modules around the plugin and the page renders unstyled **with no error**.
- **Content comes from `node_modules`, not the upstream clone.** `docs/scripts/generate-content.mjs`
  reads `.doc.mjs` from `@astryxdesign/core` and `@astryxdesign/cli`, both pinned **exact** at the
  version `packages/core` targets. Prose is reused verbatim; prop _types_ are read from

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DevRohit06/astryx-svelte](https://github.com/DevRohit06/astryx-svelte) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-15 -->
