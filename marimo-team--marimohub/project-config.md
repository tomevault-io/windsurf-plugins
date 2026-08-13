---
trigger: always_on
description: Guidance for coding agents working in this repository (a pnpm + vite-plus
---

# AGENTS.md

Guidance for coding agents working in this repository (a pnpm + vite-plus
TypeScript monorepo for marimohub). Read this before making changes.

## Build / test / lint commands

| Purpose          | Command                          | Expected on success |
| ---------------- | -------------------------------- | ------------------- |
| Install          | `pnpm install --frozen-lockfile` | exit 0              |
| Check (fmt+lint) | `pnpm check`                     | exit 0, no errors   |
| Typecheck        | `pnpm typecheck`                 | exit 0, no errors   |
| Tests            | `pnpm test`                      | all pass            |
| Coverage         | `pnpm test:coverage`             | report (v8)         |
| Build            | `pnpm build`                     | exit 0              |

The toolchain is **vite-plus** (`vp`). `pnpm check` runs `vp check`
(format + lint; it does not run the TypeScript compiler). `pnpm typecheck` runs
`vp run -r typecheck` — each package's `tsc --noEmit`. CI runs both. `pnpm test`
runs each package's `test` script (vitest); `pnpm build` builds every package.
Use these as the done-criteria for any change.

## Architecture (5 bullets)

- **Ports and adapters.** Every external dependency — storage, compute, identity
  — sits behind a TypeScript interface (a _port_). The domain depends on the
  interface, never on a vendor SDK.
- **`packages/core`** holds the domain model, services, and the port interfaces.
  It imports **no vendor SDK** — nothing that speaks to a specific provider or
  performs I/O. Its deps are generic, side-effect-free utilities only: `ulidx`,
  `zod`, `better-all`, `@opentelemetry/api` (a no-op tracing facade unless an
  entrypoint registers a provider), and the format serializers `smol-toml` and
  `yaml` (core renders `marimo.toml` and integration config files). A serializer
  is a pure function, not a vendor, so a port around it would buy no
  substitutability.
  Anything that _reaches_ something — a store, a cluster, an IdP — is a port.
- **Adapters** (`packages/storage-*`, `packages/compute-*`, `packages/auth-*`)
  implement the ports. `packages/api` wires the services to Hono/OpenAPI routes
  via `@hono/zod-openapi`.
- **`packages/config`** is the ONLY package that imports concrete adapters: it
  reads `MARIMOHUB_*` env vars, selects an adapter per `*_BACKEND` selector, and
  wires the system together.
- **Entrypoints** compose everything: `apps/server` (Node, for Docker/k8s) and
  `examples/cloudflare-worker` (Cloudflare Workers). See
  [`development_docs/architecture.md`](./development_docs/architecture.md).

## The dependency rule

Dependencies point **inward only**. `core` and `api` never import an adapter;
adapters depend on `core`'s port interfaces. `config` (and the entrypoints) are
the only places concrete adapters are imported. **Reject PRs that violate this**
— e.g. an `@marimo-hub/storage-*` / `compute-*` / `auth-*` import appearing in
`packages/core` or `packages/api`. The rule is enforced mechanically: a
`no-restricted-imports` override in `vite.config.ts` (files `packages/core/**`,
`packages/api/**`) bans `@marimo-hub/{storage,compute,auth,credentials,secrets}-*`
imports, and a colocated `package-dependencies.test.ts` in each of `core` and
`api` fails if an adapter appears in their `package.json`.

## Conventions

- **Formatting** (from `.oxfmtrc.json`): tabs for indentation, single quotes,
  semicolons, `printWidth: 100`, `trailingComma: all`. Run `pnpm check` (or
  `vp fmt`) before finishing; CI fails on unformatted files.
- **Tests** are colocated `*.test.ts` files using **vitest**, with the
  `MemoryBucket` test double imported from `@marimo-hub/core/testing`. Reusable
  conformance suites live at `@marimo-hub/core/testing/contract` (`bucketContract`,
  run by every storage adapter), `@marimo-hub/core/testing/compute-contract`
  (`computeContract`, run by the hermetic compute adapters), and
  `@marimo-hub/core/testing/browse-contract` (`browseContract`, run by every
  browsable integration kind against a live catalog — env-gated, served in CI by
  the `Catalog conformance` workflow). Result-envelope
  assertions (`expectExecResult`, `expectFileResult`) are exported from
  `@marimo-hub/core/testing` — prefer them over hand-rolled `{ success, … }` checks.
- **API response envelope** is always `{ success: true, data }` or
  `{ success: false, error: { code, message } }` (see `packages/api/src`). Sole
  exception: routes serving raw content (e.g. the notebook HTML snapshot at
  `GET …/notebooks/{nid}/html`) return the bytes directly on success; their
  errors still use the envelope.
- **Committed specs.** `packages/api/openapi.yaml` and
  `internal/schemas/{bucket,integrations}.yml` are generated from the zod
  schemas; drift tests fail the build when they are stale. Regenerate with
  `pnpm schemas:generate`. CI diffs each spec against `main` with oasdiff and
  fails on breaking changes. See `internal/schemas/README.md`.
- **Frontend** (`packages/web`) is a React 19 SPA using Tailwind v4
  (`@tailwindcss/vite`) with shadcn-style UI (`class-variance-authority`,
  `clsx`, `tailwind-merge`, `lucide-react`, `sonner`) plus
  `react-aria-components`, TanStack Query, and React Router. Helpers live in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marimo-team/marimohub](https://github.com/marimo-team/marimohub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
