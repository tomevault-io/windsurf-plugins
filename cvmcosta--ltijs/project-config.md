---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

ltijs is a TypeScript-first LTI® 1.3 tool-provider library (`Provider` class): OIDC login/launch, Deep
Linking, Assignment and Grade Services, Names and Role Provisioning, Dynamic Registration, and a
pluggable JWKS keyset endpoint. v7 is a full rewrite from the old CommonJS/Babel codebase; that legacy
source has been fully removed — everything under `src/` is TypeScript, and the package published from
`dist/` is 7.0.x. v7 ships its own types (`dist/index.d.ts`, wired through `package.json`'s `exports["."]
.types` condition); the old `@types/ltijs` DefinitelyTyped package is obsolete and being phased out
upstream. Never add it as a dependency, and never shape a type to match its old (pre-v7) definitions.
`TYPESCRIPT_PORT_PLAN.md` at the repo root is a detailed historical decision log for
*why* the architecture looks the way it does (every standing convention was arrived at through explicit
back-and-forth, documented round by round). Its "Standing conventions" section is still an accurate map of
the codebase's design rules; its "Current status" / "Directory structure" tables are stale (written
mid-port, before the `Provider` composition root and the `lti/` → flat `services/` layout settled) — trust
the actual source tree over those tables.

## Commands

```bash
npm run start                 # run from source (tsx, no build step) — src/index.ts
npm run check:format          # prettier --write
npm run check:lint            # eslint src
npm run check:tests:unit      # jest -c jest.config.ts (mocked HTTP, no real DB)
npm run check:tests:db        # jest -c jest.dbconfig.ts (real mongodb-memory-server, --runInBand)
npm run test                  # format + lint + unit tests + db tests, in that order
npm run build                 # clean dist/, tsc compile (tsconfig.build.json), copy html templates, generate dist/package.json
```

Single test file / pattern: `npx jest -c jest.config.ts path/to/file.test.ts` (or `-t "test name"`). DB
tests follow the same shape with `jest.dbconfig.ts` and match `*.dbtest.ts` only; unit tests match
`*.test.ts` only — the two configs are deliberately kept separate rather than merged, since DB tests need
`--runInBand` and a global Mongo setup/teardown that unit tests don't.

Docs site (docsify, lives in `website/`, independent from the library's own build):
`npm run docs:install` then `npm run docs:dev`.

There is no separate `lint`/`format` auto-fix script — `check:lint`/`check:format` are the only ones, and
`lint-staged` (via Husky's pre-commit hook) already runs `eslint --fix` + `prettier --write` on staged
files on every commit.

## Releasing

Bump with `npm version X.Y.Z --no-git-tag-version` (no auto-commit/tag), run `npm run test` + `npm run
build` + `npm pack ./dist --dry-run` as a final gate, commit and push, then `git tag vX.Y.Z && git push
origin vX.Y.Z`. `npm run deploy` (test + build + `npm publish ./dist`, `deploy:beta` for the `beta`
dist-tag) is the actual publish step, publishing from `dist/` rather than the repo root, see "Path
aliases" below for why. Always manual, and never run it without being explicitly asked to.

## Path aliases

Imports use `#`-prefixed Node subpath imports declared once in `package.json`'s `"imports"` field (not
`tsconfig.json` `paths`, not `tsc-alias`): `#/*`, `#services/*`, `#utils/*` (→ `src/shared/utils/*`),
`#shared/*`. Each has a `development`/`default` condition pair — `development` resolves straight to
`src/**/*.ts` (used by `tsx`, `ts-jest`/Jest's `customExportConditions`, and `tsc` via
`customConditions`), `default` resolves to the compiled `dist/**/*.js` for real `node` execution with no
custom condition active. When adding a new top-level directory under `src/`, add its alias here, not to
`tsconfig.json`.

The `development` condition must never reach a published consumer: the npm tarball only ships `dist/`, so
a consumer whose own tooling sets the `development` condition (Vitest does, unconditionally) would try to
resolve into a `src/*.ts` that was never shipped. Publishing is therefore done as `npm publish ./dist`
(not from the repo root, and not via a `publishConfig.directory` field, which is a pnpm-only convention
the npm CLI ignores) against a generated `dist/package.json` (written by
`scripts/generate-publish-manifest.js`, the last step of `npm run build`) where every `imports` entry is
collapsed to a single rebased path with no `development` key at all. The root `package.json` above stays
untouched for local dev/test/typecheck.

## Architecture

**`Provider` (`services/provider/provider.service.ts`) is the composition root.** Its constructor is the
one place that decides every default implementation (`MongoDatabaseManager`, `FetchRequestHandler`,
`ExpressHttpHandler`, `MockCacheManager`, `DefaultLogger`) and wires the concrete service graph together;
every other service takes its dependencies as required constructor parameters with **no default values**
— there is no shared/singleton instance of any service class exported from its module, only the class
itself. If you need to trace how a request actually flows end to end, start reading here.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Cvmcosta/ltijs](https://github.com/Cvmcosta/ltijs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
