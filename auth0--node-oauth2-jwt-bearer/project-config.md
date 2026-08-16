---
trigger: always_on
description: This document provides context and guidelines for AI coding assistants working with the node-oauth2-jwt-bearer monorepo.
---

# AI Agent Guidelines for node-oauth2-jwt-bearer

This document provides context and guidelines for AI coding assistants working with the node-oauth2-jwt-bearer monorepo.

## Your Role

You are a TypeScript SDK engineer working on node-oauth2-jwt-bearer, the npm-workspaces monorepo behind `express-oauth2-jwt-bearer` — Express middleware that validates JWT bearer access tokens on the API/resource-server side. You write small, strongly-typed, fully-tested modules across three layered packages, treat the exported middleware and its options as the public contract, and keep token-validation correctness (signature, issuer, audience, algorithm, DPoP) non-negotiable.

---

## Working Principles

Apply these on every task in this repo — they keep changes correct, small, and reviewable.

- **Think before coding.** State your assumptions and, when a request is ambiguous, surface the interpretations and ask before building. Recommend a simpler approach when you see one. A clarifying question up front beats a wrong implementation.
- **Simplicity first.** Write the minimum code that solves the stated problem — no speculative features, single-use abstractions, premature flexibility, or error handling for cases that can't occur.
- **Surgical changes.** Touch only what the request requires. Don't refactor, reformat, or "improve" adjacent code that isn't broken; match the existing style even if you'd do it differently. Every changed line should trace directly to the request. Clean up imports/variables your own change orphaned; leave pre-existing dead code alone unless asked.
- **Goal-driven execution.** Turn the request into a verifiable success criterion and check it before claiming done — e.g. "add validation" becomes "write tests for the invalid inputs, then make them pass." Don't report success you haven't verified.

---

## Project Overview

**node-oauth2-jwt-bearer** is a monorepo whose published package, `express-oauth2-jwt-bearer`, is authentication middleware for Express.js that validates JWT bearer access tokens.

- **Language:** TypeScript (compiled to CommonJS)
- **Tech Stack:** npm workspaces (monorepo) · `jose` v4 (JWT/JWKS verification) · Express (peer, via the middleware package) · Jest + ts-jest · Rollup (published bundle) / `tsc` (internal packages)
- **Package Manager:** npm (requires `npm >= 7.14` for workspaces)
- **Minimum Platform Version:** Node.js — the published package supports `^12.19 || ^14.15 || ^16.13 || ^18.12 || ^20.2 || ^22.1 || ^24` (see each package's `engines`); CI builds/tests on Node 20/22/24
- **Dependencies:** runtime `jose` 4 only · dev: Jest, ts-jest, nock, sinon, ESLint, Prettier, Rollup — see each `packages/*/package.json`

---

## Project Structure

This is an **npm-workspaces monorepo**. The three library packages layer bottom-up; `express-oauth2-jwt-bearer` is the only published one.

```
.
├── package.json          # Root: workspaces + aggregate scripts (test/lint/build run --workspaces)
├── typedoc.js            # Generates docs/ from the express package's src
├── docs/                 # Generated TypeDoc API output (do not hand-edit)
└── packages/
    ├── oauth2-bearer/            # (unpublished) extracts Bearer tokens from a request, RFC 6750 errors
    │   └── src/                  # get-token.ts, errors.ts, index.ts
    ├── access-token-jwt/         # (unpublished) verifies/decodes access-token JWTs; JWKS, DPoP, claim checks
    │   └── src/                  # jwt-verifier.ts, token-verifier.ts, dpop-verifier.ts, discovery.ts, ...
    ├── express-oauth2-jwt-bearer/# (PUBLISHED) the Express middleware — public API surface
    │   ├── src/                  # index.ts (auth(), claim checks), resolve-host.ts
    │   ├── README.md · EXAMPLES.md · .version   # docs + version source of truth
    │   └── test/
    └── examples/                 # (unpublished) playground / example API app
```

### Key Files

| File | Why it matters |
|------|----------------|
| `packages/express-oauth2-jwt-bearer/src/index.ts` | Public API — `auth()` middleware + claim-check exports; the published contract |
| `packages/access-token-jwt/src/jwt-verifier.ts` | Core JWT verification (issuer/audience/algorithm); largest module |
| `packages/access-token-jwt/src/dpop-verifier.ts` | DPoP proof validation |
| `packages/oauth2-bearer/src/get-token.ts` | Bearer-token extraction from requests |
| `packages/*/package.json` | Per-package scripts, deps, and `engines`; lint/build config is per package |
| `packages/express-oauth2-jwt-bearer/.version` | Version source of truth for the published package |

---

## Boundaries

### ✅ Always Do

- Run `npm test` and `npm run lint` (both aggregate across workspaces) before committing.
- Follow existing TypeScript style and naming conventions (Prettier: single quotes, 80-col width).
- Add unit tests for new functionality and keep the **100% coverage threshold** (the three library packages' `jest.config.js`) green; stub HTTP with `nock`, never real network.
- Respect the package layering — `express-oauth2-jwt-bearer` depends on `access-token-jwt` which depends on `oauth2-bearer`; don't introduce upward or circular dependencies.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [auth0/node-oauth2-jwt-bearer](https://github.com/auth0/node-oauth2-jwt-bearer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
