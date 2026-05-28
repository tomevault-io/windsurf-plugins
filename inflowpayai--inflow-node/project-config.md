---
trigger: always_on
description: Operating notes for working in this repo. If anything below conflicts with the configs (`eslint.config.js`, `typedoc.json`, `tsconfig.base.json`, `turbo.json`, `.changeset/config.json`, `.prettierrc.json`), the configs win — fix the drift, don't paper over it.
---

# AGENTS.md

Operating notes for working in this repo. If anything below conflicts with the configs (`eslint.config.js`, `typedoc.json`, `tsconfig.base.json`, `turbo.json`, `.changeset/config.json`, `.prettierrc.json`), the configs win — fix the drift, don't paper over it.

## What this repo is

A pnpm + Turborepo monorepo of InFlow's open-source Node SDKs, organized by product. Every package, example, and product doc folder takes a product prefix (`x402-`, `mpp-`, …) so products coexist without ambiguity.

- Foundation packages: `@x402/*` — declared as `peerDependencies`, never bundled.
- InFlow's published packages: `@inflowpayai/*`.
- Examples illustrate end-to-end use; they don't publish.

## Repo map

- `packages/<product>-*/` — publishable SDKs. One folder per package.
- `examples/<product>-*/` — runnable end-to-end examples. Never published.
- `docs/<product>/` — product docs (overview, architecture, wire format, extensions).
- `docs/monorepo/` — repo-level docs: `contributing.md`, `tooling.md`, `publishing.md`, `documentation.md`.
- `scripts/` — repo-level dev/CI scripts (export checks, publish verification).
- `.changeset/` — pending version bumps. PRs touching `packages/**` need an entry.

Load-bearing root files — touch with care: `tsconfig.base.json`, `turbo.json`, `eslint.config.js`, `.changeset/config.json`, `typedoc.json`, `.prettierrc.json`, `pnpm-workspace.yaml`.

## Before merging

Run all four. CI runs the same.

- `pnpm typecheck` — `tsc --noEmit` against both `tsconfig.json` (src) and `tsconfig.test.json` (src + test) per package.
- `pnpm lint` — eslint with `--max-warnings 0`.
- `pnpm test` — vitest with v8 coverage; per-package thresholds are enforced and the build fails below floor.
- `pnpm typedoc` — generates the public API reference; catches broken `{@link}` and internal-type leakage into public signatures.

Scope to one package with `pnpm --filter @inflowpayai/<name> <task>`.

## Conventions

Rules the tooling can't enforce. Breaking them lands a regression.

- **Product prefix.** Package, example folder, and product doc folder share the same `<product>-` prefix. New product means new prefix everywhere — no exceptions.
- **Peer deps stay external.** Foundation packages (`@x402/*`) are `peerDependencies` and never bundled. tsup leaves declared peers unbundled by default; don't move them into `dependencies`.
- **No `any`, no `!` non-null, no `as unknown as`** except at documented type boundaries. The canonical justified boundary cast is in `packages/x402-buyer/src/inflow-client.ts`.
- **No `console.*` in `packages/**/src/**`.** Publishable code throws typed errors and lets the caller decide what to log. `console.*` is fine in `examples/` (the output is the example) and `scripts/` (dev CLIs).
- **The package barrel is the public surface.** Anything in `src/index.ts` is public API; anything else is implementation detail.
- **`@internal` for exported-but-not-public symbols.** If a source-file export isn't re-exported from the barrel, either add `@internal` or move it into the barrel. Don't leave the question ambiguous.
- **No emoji** in code, commits, or PR descriptions unless the request explicitly calls for them.
- **No "future work" / "phase 2" / "TODO: refactor later" comments.** Describe what the code does now, or delete the comment.
- **Comments only for what the code can't say.** No restatement of behavior, no rationale-padding, no historical justification. If no non-obvious sentence comes to mind, no comment. Applies to every comment syntax — TSDoc, inline, YAML, shell, JSON-with-comments. The TSDoc rule under [Writing docs](#writing-docs) is this rule applied to one syntax.

## Node version management

Three knobs, three audiences. Don't conflate them.

- **`package.json` `engines.node`** — the floor users of the published packages need. Currently `>=22.0.0` in the root and every `packages/*/package.json`. Users see install errors when they're below this; don't bump without a real reason.
- **`.github/workflows/ci.yml` `matrix.node`** — what CI tests against. Currently `[22, 24]`. Must be a superset of `engines.node`; catches forward-compat drift one LTS cycle ahead. `release.yml` pins a single version (the active LTS, currently `24`) for publish reproducibility — that's separate from the test matrix.
- **`.nvmrc`** — what contributors use locally. Currently `22`. Read by `nvm`, `fnm`, and `volta` on directory entry. Pinned to the floor so contributors test the floor by default.

These are independent decisions. Bumping CI to test Node 26 does not bump `engines.node`. Bumping `engines.node` to drop Node 22 does not bump `.nvmrc`.

## Adding a package or product

For a new package inside an existing product: follow the template in `docs/monorepo/contributing.md` — `packages/<product>-<name>/{src,test/unit}`, `package.json` with the standard fields (`peerDependencies`, `publishConfig.access: public`, `publishConfig.provenance: true`), `tsconfig.json` + `tsconfig.test.json`, `tsup.config.ts`, `vitest.config.ts`, `README.md`. Then `pnpm install` to refresh the lockfile.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [inflowpayai/inflow-node](https://github.com/inflowpayai/inflow-node) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
