---
trigger: always_on
description: `@node-saml/node-saml` implements SAML 2.0 for Node.js. It generates `AuthnRequest`s and
---

# AGENTS.md

## What this is

`@node-saml/node-saml` implements SAML 2.0 for Node.js. It generates `AuthnRequest`s and
logout messages, and — the part that matters — decides whether an incoming SAML response
is trustworthy. It is published to npm and is the engine underneath
`@node-saml/passport-saml`, so a bug here becomes an authentication bypass in every
application downstream. Treat every change as security-relevant.

## How to read this file

This describes the library we intend to have, not uniformly the library we have today.
The codebase carries years of contributions of varying quality, and some of it predates
the standards below.

So: **this file wins over precedent.** Finding an existing pattern that contradicts a rule
here is not permission to copy it — it is a debt. When you touch such code, move it toward
the target if you can do so within the scope you were given. When you can't, leave it alone
rather than widening the change; don't let cleanup swallow the fix you were asked for.

## Layout

- `src/` — TypeScript source; the only code that ships.
- `src/index.ts` — the public barrel. Anything re-exported here is public API.
- `src/saml.ts` — the `SAML` class: option validation, request generation, and response
  validation. The security-critical path runs through `validatePostResponseAsync`.
- `src/xml.ts` — signature verification, decryption, XPath, and DOM/xml2js parsing.
- `src/crypto.ts` — PEM parsing and normalization (RFC 7468) and unique ID generation.
- `src/metadata.ts` — service provider metadata generation.
- `src/types.ts` — public types, including the whole `SamlOptions` surface.
- `test/*.spec.ts` — Mocha specs. `test/types.ts` is a shared fixture helper, not a spec.
- `test/static/` — fixtures, including `test/static/signatures/{valid,invalid}/`. See the
  warning below.
- `lib/` — build output. Generated and git-ignored; never edit.

## Commands

- `npm run build` — compile `src/` to `lib/`.
- `npm test` — runs `npm run build`, then `nyc mocha` over `test/**/*.spec.ts`.
- `npm run lint` — ESLint plus `prettier --check`.
- `npm run lint:fix` — ESLint `--fix` over `src` plus `prettier --write` over everything;
  rewrites files.

`npm test` builds first, so `npm test && npm run lint` covers everything. Run it before
calling work done.

Test order is randomized on every run (`choma`, wired up in `.mocharc.json`). A test that
depends on another test's leftover state fails intermittently rather than reproducibly, so
don't share mutable state between tests.

## Hard constraints

### Fixtures are byte-sensitive

`test/static/` contains signed SAML documents. XML canonicalization and digests depend on
the exact bytes, so re-indenting or reflowing one silently invalidates its signature, and
the resulting failure can look unrelated to what you touched.

`.prettierignore` excludes the whole directory. Keep it that way. Prettier has no XML
parser of its own today, so that entry looks redundant — it isn't. It is what makes "run
the formatter over everything" unconditionally safe, and it is what keeps an XML formatter
from silently invalidating every signature in the suite if one is ever added. That
combination is the pattern to follow: xml-crypto formats its XML with
`@prettier/plugin-xml` and keeps its signed fixtures ignored, so the plugin is welcome
here too — the ignore entry is what makes adding it safe rather than something to avoid.

When you need a new signed fixture, generate it (`docs/xml-signing-example.js` produces
the `DigestValue` and `SignatureValue`) rather than hand-editing an existing one.

Fixture names under `test/static/signatures/` encode what is signed —
`response.root-signed.assertion-unsigned.1advice-signed.xml` — and `valid/` versus
`invalid/` states the expected verdict. Follow the naming; the test names in
`test/test-signatures.spec.ts` read off it.

### The supported Node floor is real

`engines` in `package.json` is the contract, and the matrix in
`.github/workflows/workflow.yml` runs the suite on every supported version, oldest
included. Read both rather than assuming; they change. Only Node LTS versions are
supported, and `README.md` states that dropping one is a breaking change that entails a
new major. Development tooling has to install and run on the _oldest_ entry, not just the
newest.

`tsconfig.json` targets ES2018 with `lib: ["es2018"]`, and `@types/node` is pinned to the
v18 line. Anything newer than that is unavailable in `src/` even when it runs fine on your
local Node.

### The public API is semver-bound

Anything re-exported from `src/index.ts` is public. So is the option surface in
`src/types.ts`: `SamlOptions`, `SamlConfig`, and `Profile` are the configuration and
result contract that every consumer codes against. Adding a required option, renaming a
field, or tightening a default is breaking. Changes confined to `devDependencies`, tests,
CI, or tooling are not.

## Defense posture

The rules below are invariants, not aspirations, and the standard for each is the same:
**an invariant that no test can fail is not an invariant.** If you add or change one, add
the test that catches its violation, and watch that test fail before you make it pass. If

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [node-saml/node-saml](https://github.com/node-saml/node-saml) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
