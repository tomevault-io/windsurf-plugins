---
trigger: always_on
description: Notes for AI coding agents working in this repository. Humans may find the traps section useful too.
---

# Agent instructions

Notes for AI coding agents working in this repository. Humans may find the traps section useful too.

`@ajsf/*` is a JSON Schema form builder for Angular, published as five packages from one Angular CLI workspace: `@ajsf/core` plus the `@ajsf/material`, `@ajsf/bootstrap3`, `@ajsf/bootstrap4` and `@ajsf/bootstrap5` framework packages. All five version in lockstep.

## Environment

The repository targets **Angular 17.3 on Node 18.17.1** (`.nvmrc`) with TypeScript 5.4.
Read the version out of `.nvmrc` rather than typing it: it moves with each
Angular major, and an older Node fails the build with a CLI version check
rather than anything that points at the real cause.

⚠️ **`nvm use` does not stick unless nvm is sourced with `--no-use`.** Without it the shell keeps the default Node and everything still appears to work, so a build or an install silently runs on the wrong version. Start every shell that touches the toolchain with:

```bash
export NVM_DIR="$HOME/.nvm"
. "$NVM_DIR/nvm.sh" --no-use
nvm use "$(cat .nvmrc)"
```

## Commands

```bash
npm ci                       # install
npm run build:libs           # build all five packages into dist/@ajsf/
npm run build:demo           # build the libraries and the demo app
npm start                    # serve the demo
npm run test:scripts         # tests for scripts/, plain jasmine, fast
npm run changelog            # regenerate CHANGELOG.md from commit messages
```

Library tests need the headless launcher flags:

```bash
npm run test:core -- --no-watch --no-progress --browsers=ChromeHeadlessCI
```

Substitute `test:bs3`, `test:bs4`, `test:bs5`, `test:material` for the others.

## Versioning

**The package major always equals the Angular major it targets.** `@ajsf/* 16.x` is for Angular 16. Minor and patch are free.

Never edit a version by hand. One script sets the version, the internal `@ajsf/core` range and the Angular peer ranges together, and hand editing them separately is how a package ships resolving to the previous core:

```bash
npm run version:set -- 16.1.0 16      # version, then the Angular major
npm run version:set -- 16.0.0-rc.1 16 # prerelease, publishes to the next dist-tag
```

It refuses to write anything when the version is malformed or when the two arguments disagree.

## Releasing

Publishing is automated through `.github/workflows/release.yml` and npm OIDC Trusted Publishing. There is no npm token.

⚠️ **Release only when the published product changes.** A new version is for things a consumer of `@ajsf/*` can observe: widget behaviour, the public API, dependencies, supported Angular range. CI configuration, workflows, test suites, lint setup, contributor docs and internal refactors do not get a release, however large the diff. They land on `main` without touching the version and the release workflow correctly does nothing.

1. Open a PR containing only the `npm run version:set` bump.
2. Merge it. The trigger is the version **changing** in that push, so any merge that leaves it alone is a no-op. A version sitting in the repository ahead of what is on npm is fine and does not start a release.
3. The `verify` job builds and runs all five suites, ungated, and uploads `dist` as an artifact.
4. Approve the `npm-publish` deployment. Nothing reaches npm before this, and by now the build is green.
5. `release` publishes the artifact `verify` built, `core` first, then the three framework packages, and tags the commit.

The `release` job deliberately runs a **newer Node than `.nvmrc`**. It publishes prebuilt tarballs and compiles nothing, but it needs an npm recent enough for OIDC, and current npm requires Node 22 or later. Pinning it to `.nvmrc` made `npm i -g npm@latest` fail with `EBADENGINE` before any publish ran.

A version containing a hyphen goes to the `next` dist-tag, everything else to `latest`. Do not create release tags by hand: the workflow writes them, so a tag always means the version shipped.

## Coverage

Karma writes `html`, `lcov` and `text-summary` into `coverage/<project>` for all five libraries. CI uploads them to Codecov from the `20.x` matrix leg only: both legs run the same tests on the same commit, so a second upload is a duplicate.

**Codecov authenticates with the `CODECOV_TOKEN` secret, not OIDC**, which is deliberate and differs from the npm publishing flow. `codecov/codecov-action` does accept `use_oidc: true`, but the CLI has a reported failure mode where it ignores the OIDC credential, falls back to tokenless and then fails on a rate limit (`codecov-action#1461`, closed with no stated fix), and fork pull requests receive no `id-token` at all. The token is the predictable option. Do not switch this to OIDC without confirming an upload actually lands.

⚠️ **Never give the upload step `continue-on-error` or `fail_ci_if_error: false`.** It carried both from #361 to #370 and reported success on every run while Codecov rejected every upload with `Token required because branch is protected`. Nine pull requests merged before anyone noticed. A step that cannot fail cannot tell you it is broken.

## Constraints


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hamzahamidi/ajsf](https://github.com/hamzahamidi/ajsf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
