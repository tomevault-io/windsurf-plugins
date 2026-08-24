---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`node-telegram-bot-api` v2: a from-scratch, runtime-agnostic TypeScript Telegram Bot API client.
Node 18+, also runs on Bun, Deno, Cloudflare Workers, and Vercel/Deno Edge. The source is
ESM/web-standard; the **published** package is dual-module - `zshy` emits both ESM (`*.js`/`*.d.ts`)
and CJS (`*.cjs`/`*.d.cts`), exposed via the `exports` map's `import`/`require` conditions, so it can
be `import`ed or `require`d. `src/core` stays Node-free, so the edge story is unchanged. There is
**no v1 compatibility**; the v1->v2 migration cheatsheet lives in `CHANGELOG.md`.

Current development happens on `feat/v2-core`; the base/release branch is `master`.

## Commands

Bun is the primary toolchain (it is a devDependency and runs the unit suite and generator).

```sh
bun install                       # install deps (CI uses --frozen-lockfile)

npm run check                     # full local gate: typecheck (x3) + lint:core + check:edge + unit tests
npm run typecheck                 # tsc --noEmit over src (strict)
npm run typecheck:test            # tsc over test/ (needs bun types installed)
npm run lint:core                 # FAILS if src/core touches node:* or Node globals (see below)
npm run check:edge                # FAILS if src/core bundles a Node builtin (even transitively)

npm test                          # unit tests via Bun (test/unit)
npm run test:node:unit            # unit tests via Node's node:test + tsx (CI runs this on Node 22/24/26)
npm run test:e2e                  # LIVE tests against api.telegram.org (see Testing)

npm run build                     # zshy -> dist/ dual ESM+CJS (core, node, types subpaths); postbuild repairs CJS source-map refs
npm run generate:types            # regenerate generated sources from live Bot API docs
```

Run a single unit test:

```sh
bun test test/unit/transport.test.ts          # one file
bun test -t "429 then success retries"        # by test-name substring
```

## Architecture

Three source folders, one published package, exposed via subpath exports (`.`, `./node`, `./types`):

```text
src/
  core/   runtime-agnostic. Bot, Context, compose, the single Api client, Transport,
          serializeParams + encodeForm, InputFile, the markup / entity / media builders
          (optional sugar), longPoll, webhookCallback, framework webhook adapters
          (Express / Next.js), errors. Web-standard APIs only.
          -> zero node:* imports; runs on Node 18+ / Bun / Deno / Workers / edge.
  node/   the ONLY folder allowed to import node:*. Node-only sugar: fromPath() (fs
          uploads), createWebhookServer() (node:http -> delegates to core's
          webhookCallback), the managed run() polling runner, the DEBUG stderr sink.
  types/  the generated schema (re-exported by core): discriminated Update, the generated
          Api method signatures / *Params / *Result types, expanded MessageEntity.
```

The `exports` map points `.` at the core, `./node` at the Node helpers, and `./types` at the
schema; each subpath ships ESM + CJS with matching typings:

```jsonc
// package.json
"exports": {
  ".": {
    "import":  { "types": "./dist/core/index.d.ts",  "default": "./dist/core/index.js"  },
    "require": { "types": "./dist/core/index.d.cts", "default": "./dist/core/index.cjs" }
  },
  "./node":  { /* same shape -> dist/node/*  */ },
  "./types": { /* same shape -> dist/types/* */ }
}
```

`import { Bot } from "node-telegram-bot-api"` is the runtime-agnostic core; `import { fromPath }
from "node-telegram-bot-api/node"` opts into the Node helpers. The core-vs-node isolation is a CI
rule (`lint:core` + `check:edge`), not a package boundary, so the edge bundle never drags in a Node
builtin.

### The Api client

One **generated** `Api` class - no `Proxy`, no `RawApi`/`Api` split. One concrete method per Bot
API method, each a one-liner over a shared `request()`, taking a single params object plus an
optional trailing `AbortSignal` (`getMe(signal?)`, `sendMessage(params, signal?)`). Real methods
give correct stack traces, are greppable, and need no casts; adding a method is a regenerate, never
a hand-written body. `Bot` holds an `Api`; `ctx.api` and `bot.api` expose it. The whole client
ships regardless of which methods a bot calls (no per-method tree-shaking - a deliberate trade for
the single discoverable `api.*` namespace).

### Request pipeline (the only place serialization happens)

`Api.request` -> `serializeParams` (one `JSON.stringify` + `attach://` walk for nested files) ->
`encodeForm` -> `Transport` (injectable `fetch`, 429/transient retry with jittered backoff,
`{ok,result}` envelope unwrap). Structured fields (`reply_markup`, entities, `reply_parameters`,
media, ...) are **plain typed objects/arrays** that `serializeParams` stringifies once. Builders
(`InlineKeyboardBuilder`, `EntityBuilder`, `MediaGroupBuilder`, plus the sticker / profile-photo /
story builders) are **optional sugar** whose `.build()` returns the same plain shape - a literal
object works identically. `EntityType` is the typo-proof constant for entity kinds.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yagop/node-telegram-bot-api](https://github.com/yagop/node-telegram-bot-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
