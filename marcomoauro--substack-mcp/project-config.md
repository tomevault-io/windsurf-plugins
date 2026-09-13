---
trigger: always_on
description: MCP server exposing Substack automation to LLM clients. ESM, npm. Development, CI and the image
---

# CLAUDE.md

MCP server exposing Substack automation to LLM clients. ESM, npm. Development, CI and the image
run **Node 24** — `.nvmrc` and `Dockerfile` pin `24.19.0` exactly — while `engines` declares
`>=22`, so `src/` may use nothing newer than Node 22 offers.

That floor is deliberate, not inherited: 22 is the oldest Node line still receiving security
patches (18 went EOL 2025-04-30, 20 on 2026-04-30), and it is the lowest version every
production dependency accepts — `@hono/node-server` asks for `>=20`, everything else `>=18`.
Raising it breaks anyone running `npx substack-mcp@latest` on an older runtime, so it is a major
bump; the honest ceiling is the oldest supported LTS, not the version that happens to be
installed locally.

**Two CI jobs run the suite: one on `.nvmrc`, one on the floor.** The floor job derives its
version from `engines` (`node -p "require('./package.json').engines.node.match(/\d+/)[0]"`)
rather than repeating it, so the promise and the test cannot drift. Developing two majors above
the floor is precisely how an unexercised `engines` rots into a lie — the runtime differences
are real and silent, as the `fetch` stack below shows.

## Language

**Everything in the repository is written in English** — source, comments, test names,
commit messages, PR titles and descriptions, docs. This holds regardless of the language
used in the chat: do not mirror the conversation language into the codebase.

## Commands

| Command | Purpose |
|---|---|
| `npm ci` | Install from `package-lock.json` (never `npm install` in CI or Docker) |
| `npm test` | Run the suite (`node --test 'src/**/*.spec.js'`) |
| `npm run test:watch` | Same, in watch mode |
| `npm run test:coverage` | Coverage report, spec files excluded |
| `npm pack --dry-run` | Verify what ships to npm |

`npm test` runs in under a second, but **its output shape depends on the Node version**: on 24 it
is the spec reporter (tally `ℹ pass`, failures marked `✖`), on the 22 floor it is TAP — five times
noisier, tally `# pass`, failures `not ok`. Grep for both, or a perfectly green run on the wrong
version comes back empty and reads as a broken command: `grep -E '^(#|ℹ) (tests|pass|fail)'` for
the tally, `grep -E '^(not ok|✖)'` for what broke.

## Distribution and releases

Release tags publish three artifacts: the npm package through `.github/workflows/npm-publish.yml`,
the multi-architecture Docker image through `.github/workflows/docker-build-push.yml`, and metadata
for both installation methods to the official MCP Registry. The registry job is separate from the
npm job so a registry failure can be rerun without attempting to publish the same immutable npm
version twice. It waits for the independently built versioned Docker image before publishing the
combined metadata.

`package.json`'s `mcpName` and the Dockerfile label
`io.modelcontextprotocol.server.name` are ownership proofs. Both must equal
`io.github.marcomoauro/substack-mcp`; removing either makes the corresponding package fail registry
publication. `server.json` is the source metadata, but its checked-in version is not a second release
version to bump by hand: the registry job derives the top-level version, the npm package version and
the `v<version>` Docker identifier from `package.json` before publishing.

## Layout

- `src/index.js` — entrypoint only: env check, `createServer()`, stdio transport. Keep it thin.
- `src/server.js` — `createServer()` factory plus the `tools` registry. **No side effects at
  import time**: no env reads, no transport connection. Tests depend on this.
- `src/tools/<name>.js` — one file per MCP tool, exporting a zod schema and a handler.
- `src/api/substack/` — `SubstackApi` (HTTP), `SubstackPost` (ProseMirror document builder),
  `SubscriberQuery` (the subscriber filter DSL) and `image.js` (the guarded fetch of a
  caller-chosen URL, shared by `upload_image` and `update_draft`'s `cover_image`).
- `src/logger.js` — the only place that writes a log line. No dependencies, no state.
- `test/helpers/` — shared test helpers only; no tests live here.

Adding a tool means adding a file under `src/tools/` and one entry to the `tools` registry in
`src/server.js` — nothing else. `createServer()` loops over the registry calling
`McpServer.registerTool`, and the SDK derives `tools/list`, argument validation and dispatch
from what was registered, so there is no second place to update and nothing that can drift.
Do not add `setRequestHandler` calls for `tools/list` or `tools/call`: registering a tool
already covers both, and the SDK guards the clash rather than letting it slide — it throws
`A request handler for tools/call already exists, which would be overridden`.

## Substack's private API

There is no public documentation for any of this. Every endpoint below was read off the publisher
dashboard's own traffic and its `reactPublish.*.js` bundle, then confirmed against the live API —
so **check behaviour against a real request before trusting a claim about it**, including the
claims here. An official, key-authenticated Publisher API exists (`publisher_api_enabled`,
`/api/v1/publisher_api/api_key`) but is gated: the key endpoint answers **403** on a publication

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marcomoauro/substack-mcp](https://github.com/marcomoauro/substack-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
