---
trigger: always_on
description: torph is a dependency-free animated text component. pnpm workspace, published from `packages/torph`.
---

# AGENTS.md

torph is a dependency-free animated text component. pnpm workspace, published from `packages/torph`.

## Layout

| Path                                                 | What                                                                                  |
| ---------------------------------------------------- | ------------------------------------------------------------------------------------- |
| [packages/torph/](packages/torph/)                   | The library. Core engine + React / Vue / Svelte wrappers. The only published package. |
| [packages/test-cases/](packages/test-cases/)         | Shared morph corpus. Consumed by vitest _and_ the site playground.                    |
| [site/](site/)                                       | Next.js docs site — torph.lochie.me.                                                  |
| [apps/](apps/)                                       | One integration example per framework. Not published, ignored by changesets.          |
| [scripts/bundle-sizes.mjs](scripts/bundle-sizes.mjs) | Measures dist gzip sizes into the playground. Runs in `site:build`.                   |

## Commands

Run from the repo root; every script is a `--filter` into a workspace.

```
pnpm dev          # library watch build + site dev server
pnpm build        # build torph
pnpm test         # vitest, torph only
pnpm typecheck    # tsc --noEmit, torph only
pnpm lint         # torph + site
pnpm site:dev
pnpm example:react   # also :vue :svelte :svelte-ssr :typescript
```

There is no CI. `pnpm typecheck && pnpm test && pnpm lint` before anything lands.

## Code standards

### Comments

Comment only what is complex or out of the ordinary, in one line at most.

Assume a reader who knows the platform — never explain a standard API, a language feature, or what a well-named function does. `measure()` needs no comment; anyone reading it knows what measuring is. Same for WAAPI calls, `Intl.Segmenter`, React hooks.

If it won't fit on one line, it doesn't go in the code.

What earns a comment is the thing someone would otherwise "fix" — a constant that looks arbitrary, an order that looks incidental, a workaround that looks like a mistake:

```ts
const SLIDE = 20; // `offsetHeight || 20` — happy-dom has no layout
```

A worked example that proves a rule belongs in [packages/test-cases/](packages/test-cases/) and not in prose — the corpus runs, a comment's copy of it rots quietly. A rule enforced in several places is stated once, as a trailing comment on the declaration they all import — not repeated at each site, and not as a list of the sites, which goes stale on the next one added.

The exception is a file-level block explaining a whole subsystem's contract, where one already exists.

### Style

Prettier + eslint flat config, both in `packages/torph/`. Formatting is prettier's job; `eslint-config-prettier` sits last and disables anything that would fight it. Double quotes, semicolons, 2 spaces.

`no-console` is a warning — the debug path is the `debug` option, not a stray log. Unused args need a `_` prefix.

**Reach for prettier by path** — `packages/torph/node_modules/.bin/prettier` — never `npx prettier` or `node_modules/.bin/prettier`. The root resolves 2.8.8, hoisted out of `@changesets/cli`, and its `trailingComma` default of `es5` strips a comma off every multi-line call in the repo. (`pnpm --filter=torph exec prettier` finds the right binary but runs in `packages/torph/`, so paths from the root miss.)

`.prettierrc.cjs` lives in `packages/torph/`, so `packages/test-cases/` and `site/` format on prettier 3 defaults instead. The two agree on everything the repo uses.

## The library

One engine. `TextMorph` handles text and numbers both; numbers are a `kind` (`"digit" | "symbol"`) on `Segment`, on by default, opt out with the `numbers` prop.

**Segment IDs are the only identity** used for FLIP tracking and DOM reconciliation. A duplicate ID means visible text loss, because reconciliation hands one element to two segments. `createIdAllocator()` in [segment.ts](packages/torph/src/lib/text-morph/utils/segment.ts) is the single source of uniqueness for text; `unique-ids.test.ts` asserts the invariant. Don't mint text IDs outside it.

Numbers are the one exception. `mintId()` in [number.ts](packages/torph/src/lib/text-morph/utils/number.ts) issues `\u0000n<counter>`: text IDs are derived from the text, so a NULL prefix cannot collide with one, and a counter that only climbs cannot collide with an ID a number is still carrying from an earlier morph. Any third ID source has to argue for its own disjointness the same way.

**Adding a file under `src/lib/`** — `tsup.config.ts`'s `aliasCorePlugin` externalises only paths matching `../lib/text-morph`. Anything else imported from a framework wrapper gets inlined into that wrapper's bundle. This once cost the React entry 4.6 kB gzip.

New public API needs an export in [src/index.ts](packages/torph/src/index.ts) — that file is the package surface.

### Tests

vitest, no config file. Tests needing a DOM opt in per-file:

```ts
// @vitest-environment happy-dom
```

happy-dom has **no layout** — every rect is zero. Assert intent (a transform was applied, an animation was cancelled), never geometry.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lochie/torph](https://github.com/lochie/torph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
