---
trigger: always_on
description: This subtree publishes OmicsClaw to npm so `npm install -g omicsclaw` yields a
---

# npm distribution — agent guide

This subtree publishes OmicsClaw to npm so `npm install -g omicsclaw` yields a
working CLI with no Python prerequisite.

## Layout

- `omicsclaw/` — the wrapper package published as `omicsclaw`. Carries no
  runtime; ~20 KB. `bin/omicsclaw.mjs` locates the platform runtime and hands
  arguments to its interpreter.
- `build-runtime-package.mjs` — wraps a prebuilt runtime into a publishable
  `@omicsclaw/runtime-<target>` package. Does **not** build the runtime.
- `dist/` — build output, gitignored.

## The distribution model

The standard npm "thin wrapper + platform packages" pattern (esbuild, swc,
biome): the wrapper lists one `@omicsclaw/runtime-<target>` per host in
`optionalDependencies`, each declaring `os` / `cpu`. npm refuses to install
non-matching ones, and because they are *optional* that refusal is a silent
skip. Exactly one runtime lands on disk.

The runtime content itself comes from `scripts/build-backend-runtime.py` at the
repo root — python-build-standalone plus the `DESKTOP_DEPS` whitelist plus
`pip install --no-deps omicsclaw`. Do not reimplement any of that here; there
must be exactly one definition of what a runtime contains.

That script used to live in the private OmicsClaw-App repo and be checked out
over a PAT. OmicsClaw-App@faf1e16 dropped its embedded Python distribution and
deleted it, so it now lives here — which is where it belonged anyway, since the
environment it builds is defined by *this* project's dependencies.

## Hard constraints

**`npm pack` silently drops every symlink.** A PBS runtime has ~1048 of them.
The published tarball would otherwise arrive with `python/bin/python3.11` but no
`python/bin/python3`, and no `python/lib/libpython3.11.so`. Two independent
mitigations, both required:

1. `build-runtime-package.mjs` records the **dereferenced** interpreter path into
   `omicsclawRuntime.pythonRelPath` in the runtime package's manifest, and
   `resolveRuntime` prefers it. This is what makes the package work even under
   `--ignore-scripts`.
2. Every link is recorded into `runtime-symlinks.json` and recreated by
   `lib/restore-symlinks.mjs` during postinstall.

Never make resolution depend on a symlink existing.

**The postinstall entry point must be CommonJS.** `scripts/postinstall.cjs` is a
version-gated shim around `postinstall-main.mjs`. An ESM entry point is a *parse*
error on old Node, which kills the script before any `try` / `catch` and fails
the whole install with `ELIFECYCLE` — observed on npm 6.14.4 / Node 10.19.0. The
`engines` field does not prevent this; npm only warns. The dynamic `import()` is
built through `new Function` so the expression is never parsed by a runtime that
would reject it.

**The postinstall must never fail an install.** Every path exits 0. A migration
that could not run is a message, not a broken install.

**The migration must never touch what it cannot positively identify.** A shim
qualifies only when it realpath-resolves outside our package AND its content
carries a Python entry-point marker AND it is not a Node shim. Note that
`bin/omicsclaw.mjs` embeds the string
`from omicsclaw.surfaces.cli.launcher import main` in its Python bootstrap, so it
matches the marker — hence the `node_modules` counter-check in
`migrate.mjs`. `oc` is also the OpenShift client's command name; an unrecognised
`oc` is reported and left alone.

**The package must stay dependency-free.** It runs inside `npm install`, so
having dependencies of our own would be a bootstrapping problem. Tests use
stdlib `node:test` only.

## Sync contract — five places, all manual

Adding or removing a target means touching **all** of these:

1. `NPM_TARGETS` in `build-runtime-package.mjs`
2. `SUPPORTED_TARGETS` in `omicsclaw/lib/resolve-runtime.mjs`
3. `optionalDependencies` in `omicsclaw/package.json`
4. `SUPPORTED_TARGETS` in `scripts/build-backend-runtime.py`
5. the `build-runtime` matrix in `.github/workflows/npm-release.yml`

`omicsclaw/test/units.test.mjs` asserts the current list, so (2) will fail loudly
if it drifts — nothing guards the other four.

Only four of six plausible targets ship a runtime. `darwin-x64` is out because
llvmlite stopped publishing macOS x86_64 wheels; `win32-arm64` is out because
there is no native runner and no Rosetta equivalent. Both are marked
`skip-runtime: true` in the App's CI. Never package a runtime directory
containing a `SKIPPED` marker — the build script refuses, because shipping an
interpreter without `omicsclaw` in it is worse than shipping nothing.

## The descriptor contract

Postinstall writes `~/.omicsclaw/runtime.json`; OmicsClaw-App reads it in
`src/lib/npm-runtime-descriptor.ts`. Both sides validate structurally and the
schema version must move together.

npm has **no working uninstall hook for global packages** — `preuninstall` does
not fire for `npm uninstall -g` — so this file outlives the runtime it
describes. Every reader must verify `pythonPath` still exists before trusting the
entry. That check is load-bearing, not defensive garnish.

## Releasing

`.github/workflows/npm-release.yml`, manual dispatch only. Four matrix cells
build a runtime and pack one platform package each; a separate `publish` job

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TianGzlab/OmicsClaw](https://github.com/TianGzlab/OmicsClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
