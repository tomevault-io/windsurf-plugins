---
trigger: always_on
description: Instructions for AI agents working on **druk**, a terminal code editor.
---

# AGENTS.md

Instructions for AI agents working on **druk**, a terminal code editor.

`CLAUDE.md` is a symlink to this file — keep everything in here.

## What this project is

A TUI code editor built on [OpenTUI](https://github.com/anomalyco/opentui) (Solid
reconciler on a native Zig core). Shipped as a standalone binary — npm, Homebrew, a curl
installer — and run as a CLI.

Features: file tree with bulk file operations, preview/pinned tabs, tree-sitter syntax
highlighting, search (current file and project-wide), command palette, themes, vim mode,
git marks in tree/gutter/status bar, file watching with conflict prompts, per-project
session restore, and a startup update check.

## Runtime and tooling

- **Bun is required to develop** — OpenTUI's native core loads through Bun's FFI. Node
  cannot start the app from source (its `node:ffi` is not in any shipping release), so
  never "fix" a Bun dependency by switching the runtime. Users need nothing installed:
  `bun build --compile` bakes the Bun runtime, the native library and every grammar into
  one executable.
- **bun manages dependencies and scripts.** Do not use npm or pnpm for installs — the
  lockfile is `bun.lock`.
- **Say `bun run <script>`, not `bun <script>`.** `build` collides with Bun's own bundler
  subcommand, so `bun build` silently bundles nothing instead of running the script. This
  now includes `test`: bare `bun test` works, but the whole suite runs in one process and
  takes four times as long — `--parallel` lives in the script and cannot be set from
  `bunfig.toml` (the key is accepted there and silently ignored).

```bash
bun install
bun run start            # run from source, opens the current directory
bun run start ./some/dir # run from source against a directory
bun run build            # compile a binary for this machine into dist/<target>/
./dist/*/druk .          # run what you just built (bin/druk.js finds it too)
bun run build linux-x64  # …or for a named target, if its native package is installed
bun run release          # package dist/ for npm + release archives (--publish to ship)
bun run formula          # Homebrew formula for those archives (not published anywhere yet)
bun run test             # unit + UI, one worker per core (~20s; 87s without --parallel)
bun test test/foo.tsx    # a single file, where the flag buys nothing
bun run check-types      # tsc --noEmit
bun run lint             # oxlint
bun run format           # oxfmt (writes); format:check to verify
```

Always run `bun run check-types`, `bun run lint`, `bun run format` and `bun run test`
before considering a change done — `bun run check` is all four.

`--parallel` runs each *file* in its own worker process, so nothing may depend on state
shared between files. `test/setup.ts` is preloaded to give every worker its own
`XDG_CONFIG_HOME`; without it the workers fight over one `sessions.json` — and the suite
writes to your real `~/.config/druk`.

## Shipping

`bun run build` produces one executable; `bun run release` turns the executables in
`dist/` into npm packages and release archives. Five things about that are easy to break:

- **Assets must be static `with { type: 'file' }` imports.** Bun embeds only what it can
  see at build time, so a computed specifier or an `import.meta.resolve` call leaves the
  binary without that file. Every grammar and query goes through
  `src/languages/grammars.ts` for this reason.
- **The binary must not autoload `bunfig.toml`.** druk is opened inside other people's
  projects, and a standalone Bun binary otherwise reads the `bunfig.toml` it finds there —
  whose `preload` fails to resolve and kills startup. `build.ts` turns that off.
- **Cross-compiling needs the target's `@opentui/core-<platform>` package**, and
  `bun install` fetches the host's alone. That is why the release workflow uses one native
  runner per platform instead of five `--target` flags on one machine.
- **The GitHub release is uploaded before npm.** One package is published, `druk`, and it
  holds no binary: `bin/binary.mjs` fetches the archive for the machine from the release.
  Publishing npm first would leave a window where an install finds no asset.
- **There is deliberately no package per platform.** That is the usual arrangement, and
  it is what druk used to do, but creating a package needs a credential that can create
  packages — while the release authenticates as GitHub through OIDC and may only publish
  to `druk` itself. One package is what makes the release run unattended.

The repo's own `package.json` is `private`: what npm publishes is staged into
`dist/npm/druk` by `scripts/release.ts` — the shim, the postinstall and nothing else.
Versions come from `package.json` — bump it and `.github/workflows/release.yml` builds
every platform, uploads the archives to the release and publishes to npm, with no manual
step. Two ways to start it: push a tag `v<version>`, or run the workflow from the Actions
tab, which tags the commit it runs on for you.

**`package.json` is the version, not the ref.** The published shim fetches its binaries
from `releases/download/v<version>`, so the release must carry exactly that tag — the
workflow reads the version once in `check` and every later step uses it. A tag push whose

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [letstri/druk](https://github.com/letstri/druk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
