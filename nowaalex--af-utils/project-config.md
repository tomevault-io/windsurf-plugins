---
trigger: always_on
description: pnpm owns dependency installation, workspace linking, lockfile updates, and
---

# Repository conventions

## Workspace ownership

pnpm owns dependency installation, workspace linking, lockfile updates, and
publishing. Nx discovers projects from `pnpm-workspace.yaml` and owns task
scheduling, dependency ordering, and caching. Package manifests and the
filesystem are the project source of truth; do not duplicate the workspace
project list in `nx.json`.

Run repository and package tasks through Nx. A target may invoke its underlying
tool directly, such as `tsc`, `vitest`, `vite`, or `astro`, but it must express
dependencies on other targets or projects with Nx `dependsOn`. Do not use
`pnpm --filter` or nested package scripts to build an execution pipeline.

Direct pnpm commands are reserved for dependency installation, the `prepare`
Git-hook lifecycle, lockfile maintenance, environment bootstrap such as
Playwright browser installation, and Changesets version/publish boundaries.
These operations are package-manager responsibilities rather than cacheable
workspace tasks. Repository validators may query pnpm for workspace metadata,
but must not use it to schedule target pipelines.

## Commands

Use these commands from the repository root:

| Task                                     | Command                                                       |
| ---------------------------------------- | ------------------------------------------------------------- |
| Format the repository                    | `pnpm nx run workspace:format`                                |
| Check formatting and lint                | `pnpm nx run workspace:check-style`                           |
| Type-check all projects                  | `pnpm nx run-many -t typecheck`                               |
| Test all projects                        | `pnpm nx run-many -t test`                                    |
| Build publishable packages               | `pnpm nx run-many -t build --projects=tag:npm:public`         |
| Build packages and website               | `pnpm nx run-many -t build --projects=tag:npm:public,website` |
| Build standalone examples                | `pnpm nx run @af-utils/examples:build`                        |
| Type-check standalone examples           | `pnpm nx run @af-utils/examples:typecheck`                    |
| Update generated examples and versions   | `pnpm nx run @af-utils/examples:versions`                     |
| Validate generated examples and versions | `pnpm nx run @af-utils/examples:versions:check`               |
| Validate package READMEs                 | `pnpm nx run workspace:packages-readmes-check`                |
| Validate Nx task ownership               | `pnpm nx run workspace:nx-contracts-check`                    |
| Update the canonical site origin         | `pnpm nx run workspace:site-origin-sync`                      |
| Validate canonical site links            | `pnpm nx run workspace:site-origin-check`                     |
| Validate package tarballs                | `pnpm nx run-many -t publint --projects=tag:npm:public`       |
| Prepare packages for publication         | `pnpm nx run workspace:packages-ready`                        |
| Run the pre-push verification graph      | `pnpm nx run workspace:verify:prepush`                        |
| Run the complete quality graph           | `pnpm nx run workspace:verify:full`                           |
| Run browser integration tests            | `pnpm nx run @af-utils/examples:e2e`                          |
| Validate generated website files         | `pnpm nx run workspace:website-generated-check`               |
| Run stable core benchmarks               | `pnpm nx run @af-utils/virtual-core:bench`                    |
| Run every benchmark suite                | `pnpm nx run-many -t bench`                                   |
| Run the bounded mutation suite           | `pnpm nx run @af-utils/virtual-core:test:mutation`            |
| Check V8 optimization invariants         | `pnpm nx run @af-utils/virtual-core:jit:check`                |
| Enforce website performance budgets      | `pnpm nx run workspace:lighthouse`                            |

Root `package.json` scripts are convenience aliases for these Nx entry points.
CI uses the explicit Nx commands so the task graph is visible in logs. Use
`pnpm nx graph` to inspect relationships and
`pnpm nx affected -t build test typecheck` for an affected-only local check.
Repository CI intentionally runs the complete gates.

The pre-push hook runs the cacheable `workspace:verify:prepush` graph. The
hosted Quality workflow adds minimum-version compatibility, browser integration,
generated website, link, and Lighthouse checks. Its browser jobs run Chromium,
Firefox, and WebKit in parallel while the required `quality / Quality` status
remains the aggregate result.

`site.config.json` owns the production website origin. Published package
metadata and documentation must contain literal URLs, so the site-origin sync
target updates those derived copies. Change only the config value, then run the
sync target; the style gate rejects stale af-utils origins.

## Releases

GitHub accepts updates to `main` only through a pull request whose
[Quality workflow](.github/workflows/quality.yml) succeeds; the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nowaalex/af-utils](https://github.com/nowaalex/af-utils) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
