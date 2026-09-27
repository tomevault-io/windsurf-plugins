---
trigger: always_on
description: Vlak is a minimal design system built from paper, ink, gray, hairlines, and a 204px module. This file is for coding agents changing the repository. Agents installing Vlak in their own project want the consumer surfaces at the end.
---

# Vlak for agents working on this repository

Vlak is a minimal design system built from paper, ink, gray, hairlines, and a 204px module. This file is for coding agents changing the repository. Agents installing Vlak in their own project want the consumer surfaces at the end.

## Layout

```
packages/core     @noorddev/vlak        tokens (src/tokens.ts), the registry (src/registry.ts), the schema
                                          (src/schema.ts), generated CSS (css/), tokens JSON, props JSON, docs
packages/react    @noorddev/vlak-react  the components: StyleX leaves in src/components/*.tsx, precompiled at build
packages/cli      @noorddev/vlak-cli    init, add, list, search, docs, tokens; bundles the registry snapshot
packages/mcp      @noorddev/vlak-mcp    MCP server over the same snapshot (stdio and hosted Streamable HTTP)
registry/         generated               shadcn registry-item JSON, bundle.json, docs/*.md, llms.txt
apps/www          docs site               Next static export; examples under components/examples/<name>/use.tsx
```

pnpm workspace, Node 22.6 or newer. Root scripts run every package: `pnpm build`, `pnpm test`, `pnpm typecheck`.

## The single source of paint

`packages/react/src/components/*.tsx` is the only place styles are written. Everything else is generated from it and from `packages/core/src/registry.ts`:

- `packages/core/css/components/*.css`, `css/vlak.css`, `css/components.css`: from the leaves (`scripts/build-components.mjs`, `scripts/build.mjs`)
- `packages/core/css/tokens.css`, `tokens/vlak.tokens.json`: from `src/tokens.ts`
- `packages/core/props/props.json`: from the React types (`scripts/build-props.mjs`)
- `registry/*.json`, `registry/bundle.json`, `registry/docs/*`: from the registry, props.json, and the docs generator (`scripts/build-docs.mjs`, `scripts/build-registry.mjs`)

Never edit those outputs by hand. Edit the leaf, the registry, or the tokens, then rebuild. CI rebuilds and fails on any diff, so commit the regenerated files with the change that produced them.

## The rs() pairing rules

Every StyleX key in a leaf must be applied through `rs([...classes], styles.key, ...)`, so the CSS builder can project it onto an `rs-*` class (see the header of `packages/core/scripts/build-components.mjs`):

- One fixed class with one style: `rs(["rs-btn-primary"], styles.base)` becomes `.rs-btn-primary { ... }`.
- N fixed classes with N styles pair by position.
- N classes with a different number of styles become one compound selector `.a.b { ... }`.
- `cond && styles.key` pairs with the class item that shares the same condition: `rs(["rs-tab", selected && "rs-tab-active"], styles.tab, selected && styles.active)`.
- A ternary pairs branch by branch: `variant === "ghost" ? "rs-btn-ghost" : "rs-btn-primary"` with `variant === "ghost" ? styles.ghost : styles.primary`.
- Anything the pairing cannot place fails the build with a message naming the file and key. Keys applied outside `rs()` go in the `MANUAL` table of the builder; keys that are intentionally unused go in `IGNORE`.

Tokens in leaves come from `vlak` and `mq` in `packages/react/src/tokens.stylex.ts`, which alias the CSS custom properties. Add a token in `packages/core/src/tokens.ts`, its custom property in `scripts/build.mjs`, and its alias in `tokens.stylex.ts`, in that order.

## Build and test

```sh
pnpm install
pnpm --filter @noorddev/vlak build:css        # components → tokens → vlak.css → props.json
pnpm --filter @noorddev/vlak build:registry   # docs → registry JSON, bundle, llms.txt
pnpm build                                      # every package (core, react, cli, mcp)
pnpm test                                       # core integrity, react jsdom + axe, cli, mcp
pnpm typecheck
npx biome check .                               # lint; warnings are fine, errors are not
node apps/www/scripts/build-deps.mjs            # what the site build runs first
pnpm dev                                        # docs site at localhost:3000
```

Run the two core builds twice and confirm `git status` is clean the second time: generated files must be byte-stable.

## Doctrine

- Monochrome. Paper, ink, gray. No hue anywhere in the system; a test rejects hex that is not gray. Charts may carry one spot color through `spot`.
- Hairlines (1px), square corners by default, one 4px radius token for controls, the 204px module (184 column + 20 gutter).
- Sentence case. No all caps, no periods in titles, no marketing.
- Platform first: `<dialog>`, `<details>`, the Popover API, scroll snap, native inputs. JavaScript only where the platform has nothing, and then the APG pattern with full keyboard support.
- No `!important` in shipped CSS. Everything sits in cascade layers so consumers override without it.
- Accessibility is required: name, role, keyboard, focus ring, 3:1 control border, `prefers-reduced-motion`, `forced-colors`. Every interactive component has an axe test and a keyboard test.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Noord-Ventures/vlak](https://github.com/Noord-Ventures/vlak) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
