---
trigger: always_on
description: Reference for AI agents working in the `honeybadger-js` repository. Covers monorepo layout, conventions, tooling, testing, CI, and release flow.
---

# AGENTS.md

Reference for AI agents working in the `honeybadger-js` repository. Covers monorepo layout, conventions, tooling, testing, CI, and release flow.

## Repository overview

- Public Honeybadger JavaScript SDKs and build/CI plugins.
- Monorepo managed by [pnpm workspaces](https://pnpm.io/workspaces) + [Lerna](https://lerna.js.org/) (v9) in **independent** versioning mode (`lerna.json` with `"npmClient": "pnpm"`).
- All packages live under `packages/*` and publish to npm under the `@honeybadger-io/*` scope.
- Node `>= 14` for most packages (`>= 18` for `esbuild-plugin`). CI runs unit tests on Node 22 and lint/integration on Node 20; repo tooling (lerna 9) requires Node `^20.19.0 || ^22.12.0 || >=24`. Use the `packageManager` field in the root `package.json` (via Corepack) for the pinned pnpm version.
- Conventional Commits are enforced in CI via the `commitlint.yml` workflow on PR titles; release tooling derives versions and changelogs from commit messages.

## Top-level layout

```
.
├── .github/workflows/   # CI: tests, lint, commitlint, release, CDN/NPM publish
├── packages/            # All publishable packages (pnpm + Lerna workspaces)
├── scripts/             # Repo-wide helpers (e.g. clean-repo.sh)
├── pnpm-workspace.yaml  # Single source of truth for workspace package globs
├── pnpm-lock.yaml       # Lockfile (do not hand-edit)
├── lerna.json           # Independent versioning, npmClient: pnpm, release commit message
├── package.json         # Root scripts (build, test, lint, release, clean) + packageManager
├── tsconfig.base.json   # Shared TS compiler defaults (extended by packages)
├── .eslintrc            # Repo-wide ESLint config (TS + import + local rules)
├── eslint-local-rules.js# Custom rule: local-rules/no-test-imports
├── commitlint.config.js # Extends @commitlint/config-conventional
└── README.md
```

## Packages

Each package is independently versioned, has its own `package.json`, and publishes its own changelog. Inter-package deps use the `workspace:^` protocol in `dependencies`/`devDependencies` (Lerna rewrites them to real semver ranges on publish). `peerDependencies` stay as plain semver ranges.

| Package                                    | Path                          | Purpose                                                | Notable deps          |
| ------------------------------------------ | ----------------------------- | ------------------------------------------------------ | --------------------- |
| `@honeybadger-io/core`                     | `packages/core`               | Core notifier (transport-agnostic). TypeScript.        | none in-repo          |
| `@honeybadger-io/js`                       | `packages/js`                 | Universal SDK: browser + Node + AWS Lambda + Express.  | `core`                |
| `@honeybadger-io/react`                    | `packages/react`              | React error boundary + hooks.                          | peer: `js`            |
| `@honeybadger-io/vue`                      | `packages/vue`                | Vue 2/3 integration.                                   | peer: `js`            |
| `@honeybadger-io/nextjs`                   | `packages/nextjs`             | Next.js plugin (App Router + Pages Router).            | `js`, `webpack`; peer: `react` |
| `@honeybadger-io/react-native`             | `packages/react-native`       | React Native integration (iOS + Android native bits).  | `core`                |
| `@honeybadger-io/gatsby-plugin-honeybadger`| `packages/gatsby-plugin`      | Gatsby plugin (plain JS, no build step).               | `js`, `webpack`       |
| `@honeybadger-io/cloudflare`               | `packages/cloudflare`         | Cloudflare Workers integration. ESM-only.              | `core`, `js`          |
| `@honeybadger-io/webpack`                  | `packages/webpack`            | Webpack source-map upload plugin (Babel-built JS).     | `plugin-core`         |
| `@honeybadger-io/rollup-plugin`            | `packages/rollup-plugin`      | Rollup/Vite source-map upload plugin.                  | `plugin-core`         |
| `@honeybadger-io/esbuild-plugin`           | `packages/esbuild-plugin`     | esbuild source-map upload plugin.                      | `plugin-core`, `core` |
| `@honeybadger-io/plugin-core`              | `packages/plugin-core`        | Shared utilities for the build plugins above.          | none in-repo          |

Dependency rule of thumb: framework packages wrap `js`; `js` depends on `core`; build plugins share `plugin-core`.

## Standard package layout

```
packages/<name>/
├── src/              # TypeScript sources (or JS for older packages)
├── test/             # unit/  integration/  e2e/ (where applicable)
├── examples/         # Integration examples (not built or published as code)
├── dist/ or build/   # Build output (gitignored)
├── package.json
├── tsconfig.json     # extends ../../tsconfig.base.json (TS packages)
├── jest.config.js    # or jest.config.cjs
├── rollup.config.*   # if the package bundles
├── CHANGELOG.md      # generated by lerna/conventional-changelog
└── README.md
```

Source organization conventions inside `src/`:

- `index.ts` (or `<package-name>.ts`) — entry point.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [honeybadger-io/honeybadger-js](https://github.com/honeybadger-io/honeybadger-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
