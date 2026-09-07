---
trigger: always_on
description: Metabase CLI and the `@metabase/client` package it is built on. TypeScript ESM. citty + native `fetch` + Zod + @clack/prompts. oxlint + oxfmt. vitest. tsdown.
---

# CLAUDE.md

Metabase CLI and the `@metabase/client` package it is built on. TypeScript ESM. citty + native `fetch` + Zod + @clack/prompts. oxlint + oxfmt. vitest. tsdown.

This file is rules only. Architecture and rationale live in `docs/architecture.md`; workflows live in the skills under `.claude/skills/`.

## Types

- No `as` casts (`as X`, `as unknown as X`, `as never`, `as any`). Use type guards or Zod `.parse` at boundaries. `as const` is fine.
- No `any`. No `Record<string, unknown>` for API responses — every cross-network value gets a named Zod schema in the client's `domain/`, parsed at the boundary. Sole exemption: `packages/cli/src/output/projection.ts`.
- No `!` non-null assertions. Restructure with a helper returning a named result-or-`null` interface, as `packages/cli/src/core/config.ts` does with `resolveUrl` / `resolveCredential`.
- No inline object types in unions, returns, or params. Name them via `interface` or `type`.
- Derive types from values (`typeof X`, mapped types over `keyof typeof X`, citty's `ParsedArgs<typeof cmd.args>`). A hand-written interface mirroring a value's shape drifts silently.
- Type guards must check the property that distinguishes what they narrow, not a weaker shared one.

## Code

- No file extensions in imports (`./foo`, not `./foo.ts`). `import type` for type-only imports.
- No comments unless the WHY is non-obvious. Never WHAT, never task/PR/path refs. This applies to prose docs too — describe the end state, never the change to it. Delete on sight: "now", "no longer", "previously", "used to", "the new …", "moved to", "renamed".
- Fail fast at boundaries. JSON parsing, file reads, HTTP responses throw or return a typed error on malformed input. Empty `catch {}` is forbidden.
- No placeholder fallbacks for absent state. `?? ""` / `?? 0` / `?? []` / `?? {}` to satisfy a type when the semantic is "missing" hides bugs — model absence with `null` or a discriminated union.
- No magic literals. Recurring constants (byte caps, timeouts, exit codes, file modes) get a named constant colocated with the canonical user.
- No boolean traps. 2+ boolean params becomes a named-options object or two functions.
- No big inline expressions. Split into semantically-named locals; flatten ternary chains with early returns or a lookup.
- Prefer the simplest realistic expression. No ceremony — repeated `override readonly`, generic gymnastics, intermediate abstract classes — where a plain field or early return is clearer.
- When a new helper subsumes an older narrower one, delete the older one in the same change.
- An `export` in `packages/cli` needs an importer outside its own file; a `*.test.ts` importer counts. The CLI ships as a bundled binary with no `exports`/`main`/`types`, so a symbol nothing imports is exported for nobody. `packages/client` is exempt — its `exports` map makes any named module public surface.
- Never fake green. No `oxlint-disable`, no `@ts-expect-error`, no `.skip`/`.todo` on a test that used to run, no weakening an assertion to match wrong output, no silently narrowing scope. If it can't pass honestly, stop and report the blocker.

## Layout

| Path              | What                                                                                                                    |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------- |
| `packages/client` | Private Metabase API client: Zod `domain/` schemas, `resources/` methods, `http/` boundary, OAuth, version/capabilities |
| `packages/cli`    | Publishable CLI: `commands/` (shell only), `core/` (pure logic), `output/` (presentation), `runtime/` (platform glue)   |
| `tests/e2e`       | Built-binary tier against a live Metabase                                                                               |

- Nothing in `packages/client` may import from `packages/cli`, touch `process`, or mutate process-global state. Its dependency budget is `zod` (peer) + `semver` + `node:` builtins.
- Within `packages/client`, only `client.ts` and `resources/` may import from `resources/`. The CLI may `import type` from the resource subpaths the `exports` map publishes.
- `JSON.parse` only in `json.ts`. `fetch` only in `http/` (plus `packages/cli/src/core/npm-registry.ts`, which is not a Metabase endpoint). `new URL(` only in `url.ts` and `http/`. Within `packages/client`, a `setTimeout` wait loop only in `poll.ts` and `http/retry.ts`; in `packages/cli`, a hand-rolled one nowhere — wait on `node:timers/promises`.
- `process.exit` only in the CLI entry; `process.stdout.write` only in `output/`; `child_process` only in `runtime/process.ts`. `src/output/` must not import the HTTP layer — reach for `@metabase/client/errors`.
- TLS trust is the host application's to configure, so `node:tls` appears nowhere in `packages/client`; the CLI opts in for itself via `core/system-ca.ts`.
- Every `MB_` env var name is a const in `packages/cli/src/core/env.ts` and is read through `readEnv`, never raw `process.env[...]`. `MB_URL` / `MB_API_KEY` are read in `core/config.ts` only.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [metabase/metabase-cli](https://github.com/metabase/metabase-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
