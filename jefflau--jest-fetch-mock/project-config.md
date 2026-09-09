---
trigger: always_on
description: jest-fetch-mock is a fetch mock for Jest: ~1.4M npm downloads/week. It was dormant 2020–2026 (npm stuck at 3.0.3 while master accumulated fixes) and was revived in July 2026 with the 3.2.0 release. This file captures how the project works and what has been learned, so anyone — human or agent — can continue from here.
---

# AGENTS.md — working knowledge for maintainers and coding agents

jest-fetch-mock is a fetch mock for Jest: ~1.4M npm downloads/week. It was dormant 2020–2026 (npm stuck at 3.0.3 while master accumulated fixes) and was revived in July 2026 with the 3.2.0 release. This file captures how the project works and what has been learned, so anyone — human or agent — can continue from here.

## Architecture (4.x)

- `src/createFetchMock.js` — the entire implementation as a pure factory. `resolvePrimitives()` uses the environment's own `fetch/Response/Request/Headers` when **all four** exist (jest-environment-node since Jest 28); otherwise the cross-fetch fallback engages (jest-environment-jsdom). `enableMocks()` **never replaces existing globals** — it only fills gaps in fallback mode. Passthrough goes to `fetch.realFetch` (the captured original; reassignable in tests). Conditional mocking is a second mock fn (`isMocking`) routed through `configureMocking(once, matcher, body, init)`.
- `src/index.js` — the default instance bound to the global `jest` (requires Jest's injected globals at import, like 3.x). `src/factory.js` — dependency-free `createFetchMock` entry for `injectGlobals: false`. `src/setup.js` — `setupFilesAfterEnv` one-liner. `.mjs` wrappers for ESM. `package.json` has an `exports` map with a `./*` escape hatch.
- `types/index.d.ts` — **self-contained**: no `@types/jest` dependency (structural `FetchMockInstance` mirrors the jest-mock surface) and no forced `dom` lib — ambient `Response`/`Request` types come from the consumer's `lib: ["dom"]` or `@types/node` ≥ 18. **Do not reintroduce `import Global = NodeJS.Global` or `/// <reference types="jest" />`.**
- `types/test.ts` — compile-only type assertions, checked by `yarn tsc`.
- Runtime dep: `cross-fetch` only (the jsdom fallback + its passthrough). Floors: Node ≥ 18, Jest ≥ 28. The 3.x line (branch from tag `v3.2.0`) remains for older stacks and keeps the old stomp-the-globals design.

## Commands

- `yarn install && yarn test` — jest (with **enforced coverage thresholds**) + `tsc` (type tests) + `eslint`. Note eslint 10 needs Node ≥ 20.19, so the full chain wants a current Node; `yarn test:unit` (jest only) and `yarn tsc` run on anything ≥ 18, which is how the CI matrix splits it (lint has its own Node 24 job).
- `bash integration/run.sh` — packs the tarball and runs six consumer fixtures against it (see `integration/README.md`). **This is the release gate.**
- Local publishes are never done; see Releases.

## Releases

1. Update `CHANGELOG.md`, bump `package.json` version in a commit named exactly the version (e.g. `3.2.0`).
2. Tag `v<version>` and push the tag. `.github/workflows/publish.yml` re-runs the suite, verifies tag == package.json version, and publishes via **npm trusted publishing** (OIDC — no tokens anywhere; provenance is automatic). Prerelease tags (any tag containing `-`, e.g. `v3.2.0-beta.2`) go to the `next` dist-tag; stable tags to `latest`.
3. **Write a proper GitHub release for every tag** (mark betas as prereleases). Group changes, link issues/PRs, credit contributors. There were no releases 2019–2026; that must not happen again.
4. Verify after publishing: `npm view jest-fetch-mock dist-tags`, then a clean-room `npm i jest-fetch-mock@<tag>` + smoke test.
5. Gates: integration suite green before tagging; **nothing goes to `latest` without the maintainer's (Jeff Lau) explicit approval**. Betas to `next` may ship autonomously.

npm account notes: the package is npm "high-impact" (mandatory 2FA); avoid account email/2FA changes near release windows — they trigger a 72-hour publishing freeze. Sole owner: `jefflau`.

## Conventions

- Commit messages: plain imperative style ("Fix …", "Add …"), body explains why. **No AI attribution anywhere** — no AI co-author trailers or "generated with" footers in commits, PRs, releases, or issue comments.
- Credit community contributors: when adapting a stale community PR, keep them as `Co-authored-by` and thank them on the PR (e.g. #223 was landed this way after 4.6 years).
- Strict semver. The 3.x line must never break API or behavior except spec-compliance bug fixes (documented in CHANGELOG). Breaking work goes to 4.0.0 behind a beta soak on `next`.
- PRs merge with merge commits (repo convention). CI (unit matrix Node 18/20/22/24 + integration on 20/24) must be green.

## Behavioral subtleties (hard-won; tests pin all of these)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jefflau/jest-fetch-mock](https://github.com/jefflau/jest-fetch-mock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
