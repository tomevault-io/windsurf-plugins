---
trigger: always_on
description: `xml-crypto` implements XML digital signatures for Node.js. It is published to npm and
---

# AGENTS.md

## What this is

`xml-crypto` implements XML digital signatures for Node.js. It is published to npm and
depended on by security-sensitive projects such as `node-saml`, so a bug here can become
an authentication bypass downstream. Treat every change as security-relevant.

## Layout

- `src/` — TypeScript source; the only code that ships.
- `src/index.ts` — the public barrel. Anything re-exported here is public API.
- `src/signed-xml.ts` — the core signing and verification logic.
- `test/*.spec.ts` — Mocha specs.
- `test/static/`, `test/validators/` — fixtures. See the warning below.
- `lib/` — build output. Generated; never edit.

## Commands

- `npm run build` — compile `src/` to `lib/`.
- `npm test` — `nyc mocha` over `test/*.spec.ts`.
- `npm run lint` — ESLint plus `prettier --check`.
- `npm run lint:fix` — ESLint `--fix` plus `prettier --write`; rewrites files.

Run `npm run build && npm test && npm run lint` before calling work done.

## Hard constraints

### Fixtures are byte-sensitive

`test/static/` and `test/validators/` contain XML signature fixtures. Canonicalization
and digests depend on the exact bytes, so reformatting whitespace silently invalidates
signatures and the failure can look unrelated. `.prettierignore` excludes both
directories — keep it that way, and never run a formatter over them.

### The supported Node floor is real

`engines` in `package.json` is the contract, and the matrix in
`.github/workflows/ci.yml` runs the suite on every supported version, oldest included.
Read both rather than assuming; they change. Development tooling has to install and run
on the _oldest_ entry, not just the newest.

### The public API is semver-bound

Changing or removing anything re-exported from `src/index.ts`, or altering types in
`src/types.ts`, is breaking for consumers. Changes confined to `devDependencies`,
tests, CI, or tooling are not.

## Security posture

- Verification must fail closed. Never make a check more permissive to get a test green.
- Preserve constant-time comparison where it is used (HMAC verification).
- Be careful with XPath. Expressions can originate from the document under inspection;
  do not broaden what a reference is able to select.
- Prefer an explicit error over silently accepting a malformed document.

## API design

Prefer removing footguns over adding convenience. A default that makes a security
decision on the caller's behalf is a footgun: the caller lives with the consequence
without ever having made the choice.

- Don't add a default for anything security-relevant. Throw instead: the errors
  `signatureAlgorithm is required` and `digestAlgorithm is required` exist so that
  nobody silently inherits SHA-1.
- Where guessing at an extension point would be dangerous, ship an inert default rather
  than a working one. `getCertFromKeyInfo` defaults to `SignedXml.noop`, not to "trust
  the certificate embedded in the document" — that default would let an attacker supply
  the key that verifies their own signature.
- Keep dangerous-but-legitimate features off until asked for. HMAC is supported, but the
  caller has to call `enableHMAC()`.
- A choice is only real if it is documented. When you require the implementer to decide,
  list the sensible options and their trade-offs in `README.md` so they can choose
  knowingly.

## Tests

Tests should protect observable behavior rather than implementation details. Favor tests
that establish what the library accepts, rejects, emits, or considers trustworthy.
Security regressions are especially important: a test should ensure that malformed or
adversarial XML cannot cause the library to report untrusted data as valid.

- Test at a public boundary for the behavior being changed.
- Where possible, start with XML and a configuration a JavaScript caller could actually
  provide, then exercise the public API.
- Test an algorithm or utility directly only when it has an independently defined
  observable contract, such as canonicalization or exported encoding utilities. Assert its
  externally meaningful input/output behavior rather than its private implementation.
- Do not unit-test private methods merely to increase coverage or mirror their
  implementation. Uncovered code indicates either inadequately tested public behavior or
  code that may be unnecessary; determine which rather than adding private-method tests
  merely to increase coverage.
- Add a test when a change alters what the library accepts, rejects, emits, or considers
  trustworthy.
- For a bug fix, observe the regression test failing for the reported reason before
  applying the fix.

## Style

- Strict TypeScript (`strict: true`), CommonJS, target ES2020.
- Two lint rules bite often: `deprecation/deprecation` is an error, so calling a
  deprecated API fails lint even when it works; `@typescript-eslint/no-non-null-assertion`
  is an error, so no `!` assertions.
- Prettier owns formatting (`printWidth: 100`). Don't hand-format.

## Comments

Code describes itself. Name things well and keep functions small enough that the _what_
and the _how_ are readable from the code, then don't restate them in prose that goes
stale the first time someone edits the line below it.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [node-saml/xml-crypto](https://github.com/node-saml/xml-crypto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
