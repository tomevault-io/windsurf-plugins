---
trigger: always_on
description: **`devframe`** is the framework-neutral container for one devtool integration, portable across viewers. Build a single tool (its RPC, its SPA, its diagnostics, its CLI/build/spa/embedded outputs) without caring how it'll be displayed. A devframe app runs standalone (CLI, static deploy, embedded SPA) just as well as it mounts inside a hub.
---

# AGENTS GUIDE

## Positioning

**`devframe`** is the framework-neutral container for one devtool integration, portable across viewers. Build a single tool (its RPC, its SPA, its diagnostics, its CLI/build/spa/embedded outputs) without caring how it'll be displayed. A devframe app runs standalone (CLI, static deploy, embedded SPA) just as well as it mounts inside a hub.

## Stack & Structure

ESM TypeScript library. Bundled with `tsdown`. Tested with `vitest`. pnpm workspaces with catalog dependencies (`pnpm-workspace.yaml`); workspace globs reserve `playground`, `docs`, `packages/*`, `examples/*` for future additions.

Source layout:
- `src/` — library code; entry `src/index.ts`
- `test/` — vitest specs; API snapshots via `tsnapi` under `test/__snapshots__/`
- `dist/` — `tsdown` build output (committed to npm tarball via `files`)

## Development

```sh
pnpm install      # requires pnpm@10.x
pnpm build        # tsdown
pnpm dev          # tsdown --watch
pnpm test         # pnpm build && vitest (api snapshot guards against stale dist)
pnpm typecheck    # tsc --noEmit
pnpm lint --fix   # ESLint via @antfu/eslint-config
pnpm start        # tsx src/index.ts
```

The `pnpm test` script intentionally runs `build` first so `tsnapi` snapshots compare against fresh `dist/`. `tsdown-stale-guard` enforces this in `test/api-snapshot.test.ts`.

## Conventions

- RPC functions must use `defineRpcFunction`; always namespace IDs (`my-plugin:fn-name`).
- Shared state via `utils/shared-state`; keep values serializable.
- Dependencies go through the pnpm catalogs in `pnpm-workspace.yaml` (`cli`, `inlined`, `testing`, `types`) — add to a catalog and reference as `catalog:<name>`, don't pin versions in `package.json`.

### Devframe design principles

These reinforce devframe's positioning as "the container for one devtool integration, portable to multiple viewers". When in doubt, err on the side of "devframe provides primitives, the hub provides UX".

- **Single-integration scope.** Devframe describes one tool. If a feature only makes sense when multiple tools share a UI — docking, a unified command palette, cross-tool toasts, terminal aggregation — it belongs in a hub package, not here.
- **Headless by default.** No default startup banners, no opinionated logging to stdout, no default styling. Provide hooks (`onReady`, `cli.configure`, etc.); let the application print its own branding. Structured diagnostics via `logs-sdk` are fine — ad-hoc `console.log`s baked into adapters are not.
- **Mount path depends on adapter context.** Given `id: 'foo'`, the default mount path is `/__foo/` for *hosted* adapters (`vite`, `embedded`) and `/` for *standalone* adapters (`cli`, `spa`, `build`). Authors override via `DevframeDefinition.basePath`. Don't hardcode mount paths in adapter code paths that may run standalone.
- **SPAs own their basePath at runtime.** Build SPAs with relative asset paths (`vite.base: './'`); discover the effective base in the browser from the executing script's location / `document.baseURI`. `createBuild` / `createSpa` copy SPA output verbatim — no HTML rewriting, no build-time `--base` injection. The client (`connectDevframe`) resolves `.connection.json` relative to the runtime base automatically.
- **CLI flags compose from both sides.** The `cac` instance backing `createCli` is exposed both to the `DevframeDefinition` (`cli.configure(cli)`) — for capabilities contributed by the tool itself — and to the `createCli` caller — for flags added at the final assembly stage. Parsed flag values are forwarded to `setup(ctx, { flags })`. Never hardcode domain-specific flags into `createCli`.

## Structured Diagnostics (Error Codes)

All node-side warnings and errors use structured diagnostics via [`logs-sdk`](https://github.com/vercel-labs/logs-sdk). Never use raw `console.warn`, `console.error`, or `throw new Error` with ad-hoc messages in node-side code — always define a coded diagnostic.

Prefix: **`DF`**. Codes are sequential 4-digit numbers (e.g. `DF0033`). Check the existing diagnostics file to find the next available number.

### Adding a new error

1. **Define the code** in the appropriate `diagnostics.ts`:
   <!-- eslint-skip -->
   ```ts
   DF0033: {
     message: (p: { name: string }) => `Something went wrong with "${p.name}"`,
     hint: 'Optional hint for the user.',
     level: 'warn', // defaults to 'error' if omitted
   },
   ```

2. **Use the logger** at the call site:
   ```ts
   import { logger } from './diagnostics'

   // For thrown errors — always prefix with `throw` for TypeScript control flow:
   throw logger.DF0033({ name }).throw()

   // For logged warnings/errors (not thrown):
   logger.DF0033({ name }).log() // uses definition level
   logger.DF0033({ name }).warn() // override to warn
   logger.DF0033({ name }, { cause: error }).log() // attach cause
   ```

3. **Create a docs page** at `docs/errors/DF0033.md` (when `docs/` lands):
   ```md
   ---
   outline: deep
   ---
   # DF0033: Short Title

   ## Message

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devframes/devframe](https://github.com/devframes/devframe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
