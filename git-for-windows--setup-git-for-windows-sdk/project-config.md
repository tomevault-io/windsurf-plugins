---
trigger: always_on
description: This file provides guidance for AI agents and developers working with the
---

# AGENTS.md

This file provides guidance for AI agents and developers working with the
`git-for-windows/setup-git-for-windows-sdk` repository.

## Repository purpose

This repository contains a GitHub Action that sets up a Git for Windows
SDK (or a subset thereof) on a Windows runner, so that workflows can
build Git for Windows, package its installer, or run its test suite
without first having to install the full SDK by hand. The Action is
published as `git-for-windows/setup-git-for-windows-sdk` and is consumed
by [git-for-windows/git](https://github.com/git-for-windows/git),
[git-for-windows/build-extra](https://github.com/git-for-windows/build-extra),
[git-for-windows/MINGW-packages](https://github.com/git-for-windows/MINGW-packages),
[git-for-windows/MSYS2-packages](https://github.com/git-for-windows/MSYS2-packages),
and assorted other Git for Windows component repositories. Changes here
ripple across that entire ecosystem, so backwards compatibility matters.

The Action is written in TypeScript, bundled with `@vercel/ncc` into a
single `dist/index.js`, and invoked as a Node 24 action declared by
`action.yml`. There is no composite-action layer; everything happens in
`main.ts` and `src/`.

## Repository structure

- `action.yml` -- Action manifest. Declares inputs (`flavor`,
  `architecture`, `msys`, `path`, `cleanup`, `verbose`, `cache`,
  `github-token`), branding, and points `runs.main` / `runs.post` at
  `dist/index.js` under `node24`.
- `main.ts` -- Entry point. Selects between the CI-artifact fast path
  and the git-clone path, sets up `PATH`, `MSYSTEM`, and `LC_CTYPE`,
  creates `/dev/{fd,stdin,stdout,stderr}` symlinks, and handles the
  POST-action cleanup when the `cleanup` input is `true`.
- `src/git.ts` -- The git-clone path. Maps `(flavor, architecture)`
  to `(repo, artifactName)` in `getArtifactMetadata`, then either does
  a bare clone plus `git worktree add` (for `flavor: full`) or a bare
  clone plus `please.sh create-sdk-artifact` (for the other flavors,
  using `build-extra`'s `please.sh`).
- `src/ci_artifacts.ts` -- The fast path. Pulls the
  `git-sdk-<arch>-<flavor>.tar.{gz,zst}` asset from the
  `ci-artifacts` release in the matching `git-sdk-*` repo and pipes
  it through `tar.exe`. Only the `minimal` flavor (always) and
  `build-installers` (when the runner has Windows 11 24H2's
  Zstandard-capable `tar.exe`) take this path.
- `src/downloader.ts`, `src/spawn.ts` -- small utilities used by the
  two paths above.
- `src/__tests__/git.test.ts`, `__tests__/main.test.ts` -- Vitest
  tests. `main.test.ts` is gated on `RUN_NETWORK_TESTS=true` because
  it really downloads an SDK.
- `dist/index.js`, `dist/index.js.map` -- the `ncc`-bundled artifact
  that is actually executed at runtime. Always regenerated alongside
  source changes, but in a dedicated follow-up commit (see commit
  conventions below).

## Build, test, package

The full validation chain is `npm run all`, which runs in order:
`build` -> `format` -> `lint` -> `typecheck` -> `package` -> `test`.
For incremental work, the individual scripts in `package.json` are:

- `npm ci` -- install dependencies (lockfile-strict).
- `npm run build` -- `tsc`, compiling `.ts` into `lib/`.
- `npm run lint` -- `eslint **/*.ts`.
- `npm run typecheck` -- `tsc --noEmit -p tsconfig.eslint.json`.
- `npm run format` / `npm run format-check` -- Prettier.
- `npm run test` -- Vitest. Set `RUN_NETWORK_TESTS=true` to also
  exercise `__tests__/main.test.ts`, which downloads a real SDK.
- `npm run package` -- `@vercel/ncc` bundles `lib/main.js` into
  `dist/index.js` (with source map).

The `build-test` workflow (`.github/workflows/test.yml`) runs the same
chain on every PR and explicitly verifies that `dist/index.js` is up to
date by checking that `git diff -aw HEAD -- ':(exclude)dist/index.js.map'`
is empty after `npm run package`. Any push that changes `main.ts` or
`src/` must therefore also include the regenerated `dist/index.js` (in
its own follow-up commit, per the commit conventions below) before CI
sees the tip, or CI fails.

For Dependabot PRs, the `npm-run-package` workflow
(`.github/workflows/npm-run-package.yml`) does the regeneration
automatically and pushes a `npm run build && npm run package` commit
back onto the dependabot branch.

## Release model

The repository uses floating release-train tags (`v0`, `v1`, `v2`,
...) plus immutable point tags (`v2.0.0`, `v2.1.0`, ...). The
`release-tag` workflow (`.github/workflows/release-tag.yml`) is
triggered by pushing a `v*` point tag from `main`. It verifies the
tag's GPG signature against `dscho`'s published keys, creates a
GitHub Release, and fast-forwards the matching `v<N>` branch to the
tagged commit. Consumers usually pin to a major-version branch:

```yaml
- uses: git-for-windows/setup-git-for-windows-sdk@v2
```

## Input model: flavors and architectures

The Action supports four flavors of the SDK:

- `minimal` -- the smallest useful set for building and testing Git
  for Windows itself. `x86_64` only.
- `makepkg-git` -- adds enough to package `mingw-w64-git`. `x86_64`
  only (can cross-package `i686`).
- `build-installers` -- adds the tooling to assemble installers,
  Portable Git, MinGit, and friends.
- `full` -- the complete SDK as a user would install it. The only

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [git-for-windows/setup-git-for-windows-sdk](https://github.com/git-for-windows/setup-git-for-windows-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
