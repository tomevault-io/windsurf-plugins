---
trigger: always_on
description: Guidance for coding agents working in this repository. Humans should read `.github/CONTRIBUTING.md` first; this file assumes you have, and records what an agent needs beyond it: the exact commands, the invariants that are easy to break, and the checklists for the tasks that recur here.
---

# AGENTS.md

Guidance for coding agents working in this repository. Humans should read `.github/CONTRIBUTING.md` first; this file assumes you have, and records what an agent needs beyond it: the exact commands, the invariants that are easy to break, and the checklists for the tasks that recur here.

## What this repo is

`files-sdk` is a unified storage SDK for object/blob backends: one `Files` class, one `Adapter` interface, 48 adapters and 15 plugins, each published as its own subpath (`files-sdk/s3`, `files-sdk/validation`, …). It also ships a `files` CLI + MCP server, app-layer gateways for most web frameworks, and `useFiles` bindings for React/Vue/Svelte.

Bun + Turbo monorepo:

| Path | What | Published? |
| --- | --- | --- |
| `packages/files-sdk` | The SDK, CLI, gateways, plugins. `src/index.ts` (~3.8k lines) is the core. | Yes, npm `files-sdk` |
| `apps/web` | Docs + marketing site (Blume/Astro), deployed to Cloudflare Workers. Owns the docs source. | No |
| `packages/videos` | Remotion launch/release videos. | No |
| `skills/files-sdk` | The agent skill for SDK consumers. Must track user-facing changes. | Via the repo, not the npm tarball (`files` is `dist` + `docs`) |

Design intent that decides most API questions:

- **Common subset, not lowest common denominator.** Core exposes only what every adapter can do cleanly. Provider-specific features go behind `files.raw` (the native client). "Use `raw`" beats "add it to the core".
- **Fail loud, never degrade silently.** If an adapter can't honor an option (`range`, `delimiter`, `metadata`, `cacheControl`, `control`), the `Files` wrapper throws before any provider I/O, gated on the adapter's `supports*` flags. Plugins that can't enforce a guarantee fail closed.
- **Web-standard I/O.** Bodies are `Blob`/`File`/`ReadableStream`/bytes/`string`. No provider types leak into the public surface.
- **Errors are normalized** to `FilesError` with codes `NotFound | Unauthorized | Conflict | Provider` (plus the SDK-native `ReadOnly`), original error in `cause`.
- **Optional peers are never bundled and never statically imported** from a path that a consumer might take without installing them. See "Bundling".

## Commands

Install once at the root (`bun install`, Bun 1.4, hoisted linker). Then:

```sh
# Repo root (Turbo fans out to every workspace)
bun run build              # SDK: Bun bundler + tsgo .d.ts + docs copy; web: registry + blume build
bun test                   # all tests (fast, offline, mocked)
bun run types              # tsc --noEmit in every workspace (TypeScript 7 / tsgo)
bun run check              # ultracite (oxlint + oxfmt) — read only
bun run fix                # ultracite autofix; run it 2–3× until it reports clean (it is racy across threads)
bun changeset              # add a changeset (see "Changesets")

# packages/files-sdk
bun test <substring>       # path substrings, not globs: `bun test s3` also runs bun-s3, s3-fetch*, cli-conditional-s3… (not minio/r2); pass a path for one file
bun run test:coverage      # the 98% per-file gate. Only meaningful from THIS directory (root cwd = no gate)
bun run dev                # rebuild on change
bun run size               # per-subpath minified/gzipped sizes
LIVE_TESTS=1 bun test .live  # live suites against real providers (needs creds; skipped otherwise)

# apps/web
bun dev                    # docs site locally
```

Notes:

- `test`/`types` depend on `^build` in Turbo. From the package dir, `test/build-output.test.ts` runs the build itself (cold ~2 min in CI).
- Don't use `bun test --parallel` with `--coverage`; it breaks the threshold gate. Don't redirect a test run to a file (`> out.txt`); the two CLI `--stdout` tests hang on the redirected stream.
- `bun fix` rewrites files. Commit or stash before running it if you want a clean diff to review.

## Git hooks and CI

- **Pre-commit** (husky) runs `check`, `types`, `test:coverage`, and `build --filter files-sdk`. Budget a few minutes per commit. Don't bypass it with `--no-verify`; fix what it reports.
- Commits are signed via the maintainer's 1Password SSH agent. If the hook pipeline goes green and the commit then fails with `failed to write commit object`, the vault is locked. Ask the user to unlock it and re-run. Never disable signing.
- **CI** (`.github/workflows/validate.yml`) builds the SDK and runs plain `bun test` in a Node 20/22/24 + Bun matrix (the tests always execute under Bun; the Node legs only smoke-test the built package under that Node), then lints and typechecks once. CI applies no coverage threshold; the pre-commit hook is the only gate. Linux tsgo catches type errors that a macOS run with stale `node_modules` can miss; when a green local run fails in CI, read the job logs (`gh api .../jobs/<id>/logs`) rather than guessing.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [haydenbleasel/files-sdk](https://github.com/haydenbleasel/files-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
