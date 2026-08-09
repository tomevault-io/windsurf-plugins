---
trigger: always_on
description: `icp-js-core` is the monorepo for `@icp-sdk/core`, the official JavaScript/TypeScript SDK for the
---

# CLAUDE.md

## Project Overview

`icp-js-core` is the monorepo for `@icp-sdk/core`, the official JavaScript/TypeScript SDK for the
Internet Computer Protocol (ICP). It is the JS counterpart to Rust's `agent-rs`: an agent that
signs, submits, and verifies ICP requests, plus Candid serialization, identities, and principals.

The published package has **no root export** — `packages/core/src/index.ts` deliberately throws.
Everything is reached through subpath exports (`@icp-sdk/core/agent`, `/candid`, `/identity`,
`/identity/secp256k1`, `/principal`, `/agent/canister-env`).

## Essential Commands

```bash
pnpm install                              # pnpm only; npm is blocked via engines
pnpm build                                # builds every workspace package (core, migrate, e2e, docs)
pnpm test                                 # unit tests (core only); runs a build first via pretest
pnpm typecheck                            # tsc --noEmit across core, migrate, e2e
pnpm lint                                 # eslint; pnpm lint:fix to autofix
pnpm prettier:format                      # format files changed vs main
pnpm size                                 # bundle-size budgets (enforced on PRs); needs pnpm bundle
pnpm attw                                 # validates published type resolution (are-the-types-wrong)

# Targeted tests
pnpm test agent/certificate.test.ts
pnpm e2e run basic.test.ts  # requires pnpm build
```

Node version is pinned by `.node-version` (24); CI runs the matrix 20/22/24. Use `corepack enable`
to get the pinned pnpm.

See `.claude/testing.md` for the test topology, running subsets, and the PocketIC / mitm e2e setup.

Release and publishing steps live in `.github/CONTRIBUTING.md`.

## Workspace Structure

- **`packages/core`** — the `@icp-sdk/core` library. Four modules under `src/`, mapping 1:1 onto the
  subpath exports: `principal`, `candid`, `agent`, `identity` (plus `agent/canister-env`).
- **`packages/migrate`** — `@icp-sdk/core-migrate`, a codemod CLI rewriting `@dfinity/agent`-era
  imports to `@icp-sdk/core`. Versioned and released separately. No tests.
- **`e2e/node`** — Vitest e2e suite driven by PocketIC, with Motoko test canisters and a mock replica.
- **`docs`** — TypeDoc-generated API reference plus hand-written `.mdx` guides, published to the
  separate `dfinity/icp-js-sdk-docs` repo.

## Architecture

Dependency direction between core's modules is strictly one-way:
`principal` ← `candid` ← `agent` ← `identity`. The `Identity` interface itself lives in
`agent/auth.ts`, so `agent` never imports `identity`.

Each module's `index.ts` barrel is both the public API surface and a TypeDoc entry point
(`docs/typedoc.json`) — anything exported there is public and requires JSDoc.

### `principal/`

`Principal` — an ICP principal, stored as bytes, rendered as base32-with-CRC32 text. Also exports
`base32Encode`/`base32Decode`, `getCrc32`. Depends on nothing else in the package.

Per repo convention, code holds the byte encoding but comments/docs use the text encoding.

### `candid/`

- `idl.ts` (~2.6k lines) — the `IDL` namespace: type constructors (`IDL.Record`, `IDL.Variant`,
  `IDL.Text`, …), `IDL.encode`/`IDL.decode`, and the visitor machinery in `candid-core.ts`.
  Analogous to Rust's `candid` crate, but a *runtime* type system — type descriptors are values you
  build and pass around, not derive macros.
- `candid-ui.ts` — renders an IDL type into form/UI descriptors.
- `utils/` — `PipeArrayBuffer` (the read/write cursor used throughout de/serialization), LEB128
  codecs, the Candid field-name hash, byte helpers.

### HttpAgent

- `agent/api.ts` — the `Agent` interface (`call`, `update`, `query`, `readState`, `status`,
  `fetchRootKey`, `getPrincipal`, optional `invalidateIdentity`/`replaceIdentity`) and its request and
  response types.
- `agent/http/index.ts` (~1.8k lines) — `HttpAgent`, the only real implementation. Construct with
  `HttpAgent.create()` (async; honours `shouldFetchRootKey`/`shouldSyncTime`) or `createSync()`.
  Owns host resolution (`determineHost`), the hard-coded mainnet `IC_ROOT_KEY`, retry/backoff,
  query-signature verification against cached subnet node keys, clock-drift tracking (`syncTime`,
  `getTimeDiffMsecs`), and the request transform pipeline.
- `agent/actor.ts` is the ergonomic layer: `Actor.createActor(idlFactory, config)` builds a typed
  `ActorSubclass` whose methods encode/decode Candid and dispatch through the agent.

### `identity/`

Implementations of the `SignIdentity`/`Identity` contract from `agent/auth.ts`:

- `ed25519.ts`, `ecdsa.ts` (WebCrypto), `webauthn.ts`, and `secp256k1/` — a separate subpath export
  because it pulls in `@scure/bip32`/`bip39` and a larger curve implementation.
- `delegation.ts` — `DelegationChain`/`DelegationIdentity`, including the `permissions` field wired
  through in v6. Normally produced by the separate `@icp-sdk/auth` package.
- `partial.ts` — a public-key-only identity that cannot sign.
- `attributes.ts` — `AttributesIdentity`, a decorator injecting signed `sender_info` into request
  content before the inner identity signs (skipped for `read_state`).

### Build outputs


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dfinity/icp-js-core](https://github.com/dfinity/icp-js-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
