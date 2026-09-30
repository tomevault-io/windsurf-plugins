---
trigger: always_on
description: Instructions for AI coding agents working in this repository. Read the whole file before
---

# AGENTS.md

Instructions for AI coding agents working in this repository. Read the whole file before
changing code. Humans: see [README.md](./README.md). Policy on AI use: [AI_POLICY.md](./AI_POLICY.md).

## 1. What this repository is

`@clickhouse/click-ui` is ClickHouse's React component library (design system).

- It is **public on GitHub** and used by many ClickHouse apps. A change to a prop, a style,
  or an ARIA attribute ships to all of them.
- Stack: React 18+, TypeScript 5+ (`strict`), CSS Modules, `cva` + `clsx`, Vite, Storybook,
  vitest + Testing Library, Playwright (visual tests, local + CI), Chromatic (visual, CI).
- Tooling: Yarn 4 through corepack (`packageManager` in `package.json`), Node `>=22.12.0`.
- Output: unbundled ESM + CJS with one `.module.css` per component. Consumers bundle it.
  Keep exports tree-shakeable: no side effects at module top level.
- `styled-components` was removed. Do not add it back. A few files still have `$`-prefixed
  internal props left over from that migration. Do not copy that naming into new code.

## 2. Commands

```sh
yarn                            # install
yarn dev                        # Storybook on http://localhost:6006 — the only dev playground
yarn test                       # vitest; filter by name: yarn test Button
yarn typecheck                  # tsc --noEmit
yarn lint                       # eslint (src, plugins) + stylelint (src/**/*.css); yarn lint:fix
yarn format                     # prettier check; yarn format:fix writes
yarn lint:code --prune-suppressions  # after fixing a frozen jsx-a11y violation (see section 7)
yarn circular-dependency:check
yarn build                      # .scripts/bash/build_pkg_dist -> dist/
yarn storybook:build            # static Storybook -> .storybook/out (what CI deploys)
yarn test:visual <Name>         # Playwright visual tests in Docker (Linux); needs Docker running
yarn test:visual:update <Name>  # regenerate snapshots — only when the visual change is intended
yarn test:visual names          # list names you can pass to the two commands above
yarn changeset:add              # interactive wizard; see section 9 for writing the file by hand
yarn generate:tokens            # rebuild src/theme tokens from tokens/themes; token owners only, see section 4
```

CI runs these as separate checks: `typecheck`, `lint`, `format`, `circular-dependency:check`,
`test`, `build` + `build:health_check`, visual regression (only specs whose `@covers` line
matches the diff), Chromatic, and a PR-title check (Conventional Commits).

Before you open a PR, run all of this and make it pass:

```sh
yarn typecheck && yarn lint && yarn format && yarn circular-dependency:check && yarn test && yarn build && yarn storybook:build
```

## 3. Repository map

```
src/
  components/<Name>/
    <Name>.tsx            # implementation
    <Name>.types.ts       # exported prop types
    <Name>.module.css     # styles (BEM class names, tokens as CSS variables)
    <Name>.test.tsx       # vitest + Testing Library
    <Name>.stories.tsx    # Storybook — the usage documentation
    index.ts              # public exports of this component
  index.ts                # public barrel — never import from it inside the library
  theme/                  # tokens in theme/tokens/*.ts and theme/styles/tokens-*.css
  lib/cva.ts              # exports `cva` (class-variance-authority) and `cn` (clsx)
  utils/test-utils.tsx    # exports `renderCUI()` — use it in every unit test
  hooks/ providers/ types/ assets/
tests/<family>/<name>.spec.ts   # Playwright visual specs, grouped by family (buttons, cards, forms, overlays, display, ...)
tests/utils/                    # shared test helpers only (getStoryUrl). Never put specs here.
tokens/themes/                  # Figma export (tokens-studio); source of src/theme/tokens/*.ts and theme/styles/tokens-*.css. Never hand-edit, see section 4
plugins/css-colocate/           # Vite + PostCSS plugins: CSS colocation and the `clickui` cascade layer
.changeset/                     # pending changesets
.claude/skills/                 # step-by-step procedures (see section 11)
```

## 4. Rules that must not be broken

1. **Never weaken a check to make CI green.** Do not relax assertions, delete or skip tests,
   grow `eslint-suppressions.json`, or regenerate visual snapshots unless the visual change is
   confirmed intended. A lint or type disable always carries a reason:
   `// eslint-disable-next-line <rule> -- <reason>`.
2. **No new runtime dependency** without a written reason in the PR. Look in `src/utils` and
   `src/lib` first.
3. **Public API changes need a changeset** (new component, prop, variant, changed default,
   removed anything). Until v1.0.0 the bump is `patch` or `minor`, never `major`; a breaking
   change is a `minor` with a migration note. A new component starts with a Component RFC:
   `gh pr create --template component_rfc.md`.
4. **This repo is public.** Never put internal links (Linear, Notion, internal Slack, etc.)
   in code, comments, commits, changesets, or PR text. A bare ticket ID such as `CUI-123` is fine.
5. **Keep refactors pure.** A styling or structural refactor must not change behavior, DOM

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ClickHouse/click-ui](https://github.com/ClickHouse/click-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
