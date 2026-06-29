---
trigger: always_on
description: This file provides guidance to coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to coding agents when working with code in this repository.

## Repository Overview

`kubernetes-models-ts` is a pnpm + Turbo monorepo that generates TypeScript models (with JSON-schema runtime validation) for Kubernetes core APIs and a large catalog of third-party CRDs. Most published packages are not hand-written: their `gen/` and `dist/` trees are produced by code generators in `utils/`.

## Workspace Layout

`pnpm-workspace.yaml` defines five workspace roots; each has a distinct role:

- `core/` — runtime libraries shipped to consumers.
  - `core/base` (`@kubernetes-models/base`): `Model` base class and resource registry (`src/model.ts`, `src/class.ts`).
  - `core/validate` (`@kubernetes-models/validate`): Ajv-based validator wired into every generated model.
- `first-party/` — Kubernetes API models.
  - `first-party/apimachinery`: shared `apimachinery` types.
  - `first-party/kubernetes-models`: top-level `kubernetes-models` package; built by `scripts/generate.ts` against the upstream OpenAPI schema using `@kubernetes-models/openapi-generate`.
- `third-party/` — one package per third-party CRD set (Argo, Cilium, Istio, …). Most have a `crd-generate.input` array of CRD URLs/paths in `package.json` and are built by `crd-generate`; a few (`prometheus-operator`, `gateway-api`) use `go-generate` instead — see "Generating from Go sources" below.
- `utils/` — generators and helpers used at build time.
  - `crd-generate`, `openapi-generate`: the two code generators that emit `gen/`.
  - `generate`: shared types/utilities the generators import.
  - `read-input`, `string-util`: small helpers.
- `internal/` — private (unpublished) tooling.
  - `publish-scripts`: the `publish-scripts build` / `publish-scripts prepack` CLI used by generated model packages (see "Generated model build pipeline").
  - `diff-crd-inputs`: dev tool for diffing CRD input lists.
  - `go-generate`: alternative model generator (`go-generate` CLI) that extracts schemata from upstream Go API packages instead of CRD YAML. See "Generating from Go sources".

## Common Commands

```sh
pnpm install                 # bootstrap (Node >=22, pnpm >=11 required)
pnpm run build               # turbo build across workspace (concurrency 4)
pnpm run build --filter <pkg>    # build a single package (Turbo --filter)
pnpm run build --force           # force rebuild, bypass Turbo cache
pnpm run clean               # remove dist/, gen/, *.tsbuildinfo from every package
pnpm test                    # run vitest across the repo
pnpm vitest path/to/file     # run a single test file
pnpm vitest -t "name"        # run tests matching a name
pnpm run lint                # eslint + syncpack
pnpm exec syncpack fix-mismatches   # align cross-workspace dep versions
(cd internal/go-generate/go && golangci-lint run ./...)   # lint Go sources
pnpm changeset               # add a changelog entry (Conventional Commits style)
```

Always build from the repo root via `pnpm run build` so Turbo orders dependencies correctly; pass `--filter <pkg>` to scope to one package and `--force` to bypass the Turbo cache. A typical third-party package's `build` script is `crd-generate && publish-scripts build`; packages on go-generate use `go-generate && publish-scripts build`; `kubernetes-models` runs `node scripts/generate.ts && publish-scripts build` instead.

### Adding a new third-party CRD package

```sh
pnpm run new-crd-package --name '<pkg>' --description '...' --author 'Name <email>'
pnpm install
# edit third-party/<pkg>/package.json: fill crd-generate.input with CRD URLs/paths
pnpm run build --filter @kubernetes-models/<pkg>
# add README.md and __tests__/ (see existing packages)
```

`scripts/new-crd-package.ts` scaffolds `package.json`, `tsconfig.json`, and a changeset. It only creates packages under `third-party/`.

### Updating a third-party CRD package

For each package, work one at a time and only commit if every step succeeds:

1. Find the latest upstream tag (e.g. `gh api repos/<owner>/<repo>/tags --jq '.[].name'` filtered for the relevant prefix).
2. Edit the version pin(s) in `third-party/<pkg>/package.json` `crd-generate.input`. Keep historical anchor URLs that intentionally pin earlier API versions (see e.g. `cert-manager`, `contour`).
3. `pnpm run build` from the repo root (so generated `gen/` is up to date) and `pnpm run lint`.
4. `pnpm test --run third-party/<pkg>` — if test fixtures fail because the upstream schema got stricter (new required fields, regex patterns, etc.), the bump is breaking; revert and skip.
5. Run `pnpm diff-crd-inputs --base master --package third-party/<pkg>`. The tool reports any CRD kinds/versions that exist on `master` but not in the working tree. If it reports removals, **add the new YAML URL and keep only the specific old YAML files that contain the removed kinds/versions** — drop any old URLs whose kinds are still covered by the new release. Re-run the build/test/diff after appending.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tommy351/kubernetes-models-ts](https://github.com/tommy351/kubernetes-models-ts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
