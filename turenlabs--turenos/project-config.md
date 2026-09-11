---
trigger: always_on
description: - To regenerate the legacy JavaScript SDK, run `./packages/sdk/js/script/build.ts`.
---

- To regenerate the legacy JavaScript SDK, run `./packages/sdk/js/script/build.ts`.
- After changing the public Protocol or Server `HttpApi`, run `bun run generate` from `packages/client`. Do not edit `src/generated` or `src/generated-effect` directly.
- Keep runtime dependencies directed from Schema to Core and Protocol, then from Core and Protocol to Server. Client runtime code may depend on Schema and Protocol but never Core or Server; `sdk-next` composes Client, Core, and Server.
- The default branch in this repo is `dev`.
- Local `main` ref may not exist; use `dev` or `origin/dev` for diffs.

## Releases

- Use the committed orchestration in `docs/release-automation.md`. Do not recreate manual artifact-copy or public-mirror procedures from old session notes.
- Prepare and merge the version change with normal CI first. From clean, synchronized `dev`, run `./script/release <version>`; GitHub Actions builds/signs privately, verifies and publishes the public release, then updates Homebrew.
- If the private release already exists, use `./script/release <version> --publish-existing`. Never rebuild/re-sign a published version or overwrite published assets.
- Keep build/signing credentials in `turenio/turen`. `PUBLIC_RELEASE_TOKEN` is a dedicated fine-grained token for `turenlabs/turenos` and `turenlabs/homebrew-turenos`; never upload a developer's general-purpose local login token.
- Preserve the pinned signing identity, exact six source exclusions, public-only Git ancestry, non-force pushes, draft verification, and downgrade/concurrency guards.
- A release is complete only after the workflow's public release, anonymous update-feed checks, and Homebrew verification succeed. Prefer quiet periodic status checks over streaming workflow logs.

## Dev Builds

- Packaged dev app (unsigned, dev channel, `com.turenlabs.forge.dev` data): from `packages/desktop`, run `bun run build && bunx electron-builder --mac dir --config electron-builder.config.ts --publish never "--config.mac.identity=-" "--config.mac.notarize=false"`, then open `dist/mac-arm64/TurenOS Dev.app`. For renderer-only changes, skip `prebuild` and the verify scripts: `bunx electron-vite build` followed by the same `electron-builder` command.
- Shells spawned inside another Electron app inherit `ELECTRON_RUN_AS_NODE=1`, which makes any `electron` binary run as plain Node and exit silently. Prefix Electron launches and `electron-vite dev` with `env -u ELECTRON_RUN_AS_NODE`.
- Headless `serve` requires `FORGE_SECRET_VAULT_KEY_ID` plus a base64 32-byte `FORGE_SECRET_VAULT_KEY`; a key that did not seal existing credentials fails startup with "Stored credentials belong to another OS-protected key". For a throwaway instance, point `XDG_DATA_HOME`, `XDG_CONFIG_HOME`, `XDG_STATE_HOME`, and `XDG_CACHE_HOME` at a scratch dir.

## Branch Names

Use a short branch name of at most three words, separated by hyphens. Do not use slashes or type prefixes such as `feat/` or `fix/`.

Examples: `session-recovery`, `fix-scroll-state`, `regenerate-sdk`.

## Commits and PR Titles

Use conventional commit-style messages and PR titles: `type(scope): summary`.

Valid types are `feat`, `fix`, `docs`, `chore`, `refactor`, and `test`. Scopes are optional; use the affected package or area when helpful, e.g. `core`, `forge`, `app`, `desktop`, `sdk`, or `plugin`.

Examples: `fix(desktop): preserve window state`, `docs: update contributing guide`, `chore(sdk): regenerate types`.

## Style Guide

### General Principles

- Keep things in one function unless composable or reusable
- Do not extract single-use helpers preemptively. Inline the logic at the call site unless the helper is reused, hides a genuinely complex boundary, or has a clear independent name that improves the caller.
- Avoid `try`/`catch` where possible
- Avoid using the `any` type
- Use Bun APIs when possible, like `Bun.file()`
- Rely on type inference when possible; avoid explicit type annotations or interfaces unless necessary for exports or clarity
- Prefer functional array methods (flatMap, filter, map) over for loops; use type guards on filter to maintain type inference downstream
- In `src/config`, follow the existing self-export pattern at the top of the file (for example `export * as ConfigAgent from "./agent"`) when adding a new config module.
- In Effect generators, bind services to named variables before calling methods. Do not use nested service yields such as `yield* (yield* Foo.Service).bar()`.

Reduce total variable count by inlining when a value is only used once.

```ts
// Good
const journal = await Bun.file(path.join(dir, "journal.json")).json()

// Bad
const journalPath = path.join(dir, "journal.json")
const journal = await Bun.file(journalPath).json()
```

### Destructuring

Avoid unnecessary destructuring. Use dot notation to preserve context.

```ts
// Good
obj.a
obj.b

// Bad
const { a, b } = obj
```

### Imports

- Never alias imports. Do not use `import { foo as bar } from "..."` or renamed imports like `resolve as pathResolve`.
- Never use star imports. Do not use `import * as Foo from "..."` or `import type * as Foo from "..."`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [turenlabs/turenos](https://github.com/turenlabs/turenos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
