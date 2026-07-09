---
trigger: always_on
description: A Yarn 4 monorepo publishing two ESM TypeScript packages:
---

# Proof VC - AI Assistant Guide

A Yarn 4 monorepo publishing two ESM TypeScript packages:

- **`@proof.com/proof-vc-common`** (`packages/common`) — public/frontend, runs in the browser **and** Node, **zero runtime deps**. Builds OID4VP Authorization Request URLs (`createClient` / `buildAuthorizationUrl`) and reads the `vp_token` from the redirect (`parseAuthorizationResponse`). No secrets, no `nonce` generation, no transaction data.
- **`@proof.com/proof-vc-server`** (`packages/server`) — privileged/backend, **Node only**. Adds `verify` / `verifyVPToken` (SD-JWT-VC verification, X.509 chain validation against an embedded trust root), Pushed Authorization Requests, transaction data, and DC API requests. Depends on `proof-vc-common` and **re-exports it**, so backend integrators need one import. Yarn links it locally via transparent workspaces.

`server` reuses `common`'s URL/param builders via the `@proof.com/proof-vc-common/internal` subpath export (server-only; not public frontend surface).

## Hard Rules

1. **`packages/common` must stay pure.** No `node:*`, `@sd-jwt/*`, `@owf/*`, or other runtime imports from anything reachable from `packages/common/src/index.ts` or `packages/common/src/internal.ts`. Type-only imports (`import type` / `export type *`) are safe — `verbatimModuleSyntax: true` erases them. Verify after build (see Package Boundaries).
2. **Prompt before publishing.** Never bump version, push tags, create a Release, or trigger the publish workflow without explicit confirmation. Publishes are permanent.
3. **Run `yarn check-all` before any commit or push.** Composes format, lint, typecheck, publint.
4. **Keep `yarn publint` on `--pack npm`.** `--pack auto` picks yarn-1 mode and reports false-positive errors.
5. **Keep `engines.node` at `>=22.0.0` and keep the CI `test-matrix` covering that floor.** Node 22 is the oldest maintained LTS (Node 20 is EOL; `@sd-jwt/*` needs 20+). `>=22` is a lower bound, so it still allows 24 and newer. The floor is checked at runtime by the `test-matrix` job (`yarn test` on Node 22 and 24); `@types/node` tracks the dev runtime (24, from `.node-version`), not the floor. Invariant: the `test-matrix` low entry must equal the `engines.node` floor, so raise the floor by bumping both together. If you drop the matrix, pin `@types/node` to the floor major so typecheck guards it instead.
6. **Never use `eslint-disable` as a workaround.** If a lint rule fires, fix the underlying code or surface the rule to the user for a config decision — do not silence it inline. Same applies to `@ts-ignore` / `@ts-expect-error` and other suppression comments.

## Package Boundaries

Versioning is **lockstep**: both packages always share one version, published together from one GitHub Release.

`packages/common/src` (browser + Node, zero deps):

- `index.ts` (public entry), `internal.ts` (server-only helpers)
- `client.ts` — `createClient`, `buildAuthorizationUrl`, `parseAuthorizationResponse`, and the shared param/URL builders
- `dcql.ts`, `constants.ts`, `types.ts` (shared wire types — the single source of truth; server imports & re-exports them)

`packages/server/src` (Node only):

- `index.ts` — `export * from "@proof.com/proof-vc-common"` then the server surface (its `createClient` intentionally shadows the frontend one)
- `client.ts` — server `createClient` (PAR, `transactionData`, `dcApiRequest`); `authorizationUrl` is `async` (PAR fetches)
- `verifier.ts` — `createVerifier` → `verify` / `verifyVPToken`
- `transaction_data.ts`, `proof_credentials.ts`, `proof_credential_factory.ts`, `utils.ts`, `types.ts` (`ProofCredential`/`VPToken`/`TrustRoot`), `certificates/**`

Verify `common` stays free of runtime leaks after build:

```bash
grep -lE '(jose|@sd-jwt|@owf|node:)' packages/common/dist/*.js
# Must match nothing.
```

## Essential Commands

Run from the repo root. Use `corepack yarn …` (or just `yarn`, with Corepack enabled).

| Command             | Purpose                                                                            |
| ------------------- | ---------------------------------------------------------------------------------- |
| `yarn check-all`    | Full check: format, lint, build, publint                                           |
| `yarn build`        | `tsc -b` (project references; builds common then server; errors on any type error) |
| `yarn test`         | `yarn workspaces foreach` run tests (each self-builds)                             |
| `yarn lint:check`   | eslint, no fix                                                                     |
| `yarn lint`         | `eslint --fix`                                                                     |
| `yarn format:check` | `prettier --check`                                                                 |
| `yarn format`       | `prettier --write`                                                                 |
| `yarn publint`      | publint over both publishable packages (`--pack npm`)                              |

Installs are immutable by default (`.yarnrc.yml`). When changing dependencies or workspaces, run `yarn install --no-immutable` to update `yarn.lock`, then commit it.

## Package manager (Yarn for dev, npm only for release)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [proof/proof-vc-common](https://github.com/proof/proof-vc-common) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
