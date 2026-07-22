---
trigger: always_on
description: This is the canonical shared instruction file for coding agents working in this repository.
---

# AGENTS.md

This is the canonical shared instruction file for coding agents working in this repository.

If another agent-specific file exists, it should import or defer to this file for shared repository guidance.

## Project Map

Egg is maintained as a utoo monorepo.

- `packages/` contains core framework packages and shared internals.
- `plugins/` contains optional Egg integrations.
- `tools/` contains developer tooling such as CLI packages.
- `tegg/` contains the tegg ecosystem.
- `examples/` contains sample applications.
- `site/docs/` contains the English and Chinese documentation site.
- tests usually live beside packages under `test/`, often with fixtures under `test/fixtures/`.

## Core Commands

The repository runs on [utoo](https://github.com/utooland/utoo) (`ut`); the workspace is still defined in `pnpm-workspace.yaml` (catalog mode), so `ut install` reads it via `--from pnpm`.

- `corepack enable utoo` enables utoo on a clean machine.
- `ut install --from pnpm` hydrates the workspace.
- `ut run build` builds all packages.
- `ut run test` runs the main test suite.
- `ut run lint` runs linting.
- `ut run typecheck` runs TypeScript checking.
- use filtered commands for focused work, for example `ut run test --workspace @eggjs/bin` or `ut run build -- --workspace ./tools/egg-bin`.

### Local CI

Run tests **without building first**. The CI workflow (`ut install --from pnpm → ut run ci`) never runs `build` before tests. If `dist/` directories exist from a prior build, tegg plugin tests will fail with `duplicate proto` errors because globby scans both `src/*.ts` and `dist/*.js`, loading the same decorated class twice.

When you see `duplicate proto` failures locally:

```bash
find tegg packages plugins tools -name dist -type d \
  -not -path '*/node_modules/*' -not -path '*/test/*' -not -path '*/fixtures/*' \
  -exec rm -rf {} +
```

Then re-run tests.

## Coding Conventions

- prefer existing repo patterns over inventing new ones
- prefer ESM and TypeScript-first changes where applicable
- keep file names lowercase with hyphens
- keep public API changes deliberate and documented
- use `oxfmt` and `oxlint --type-aware` conventions already present in the repo
- **tegg multi-app isolation**: do NOT introduce new process-global mutable
  runtime state in `tegg/`; per-app state must be backed by a `TeggScope` slot.
  Hooks registered through the bag-pinned `app.*LifecycleUtil` getters need no
  extra wrap; detached/escape-point access (timers, emitter listeners, proxy
  handlers, module-level lifecycle-util statics) must run inside
  `TeggScope.run(app._teggScopeBag, ...)`. See the "Multi-App Isolation
  (TeggScope)" section in `tegg/CLAUDE.md` for the full rules.
- **V8 startup snapshot dependencies**: the egg-bundler can build a V8 startup
  snapshot (`snapshot: true`), where the app boots only to `configWillLoad` at
  BUILD time. Any module loaded or instantiated during that boot that creates a
  non-serializable native binding — llhttp `HTTPParser` (http/https/undici),
  `nghttp2` (http2, and anything built on it), tls `SecureContext`, dns
  `ChannelWrap`, a `WebAssembly` instance (undici's llhttp; WASM is disabled under
  `--build-snapshot`), fs watchers, native addons, open sockets — makes the
  snapshot build FATAL ("global handle not serialized"). Such modules must be kept
  EXTERNAL (not inlined) so the prelude stubs them at build and forwards to the
  real module via `globalThis.__RUNTIME_REQUIRE` at restore. The framework default
  list is `DEFAULT_SNAPSHOT_LAZY_MODULES` in `tools/egg-bundler/src/lib/prelude.ts`
  (network builtins + `inspector` + `undici` + `urllib`); apps extend it via
  `egg.snapshot.lazyModules` in `package.json`. **When adding a framework
  dependency that touches the network/native stack during boot, check whether it
  must be added to that list.** A package that only reaches the network stack
  _transitively_ is already covered because those builtins are lazy (e.g.
  `@modelcontextprotocol/sdk` → `@hono/node-server` → `http2`, `@grpc/grpc-js` →
  `http2`); only a package that DIRECTLY creates native/WASM state at module-eval
  or boot-time instantiation (like `undici`) needs adding. See the "Snapshot
  lazy-external defaults" section in `wiki/packages/egg-bundler.md` for details.

## TypeScript Global Types

- put package-wide global augmentations in a dedicated `src/global.ts` or `src/global.d.ts`
- shared cross-package global types belong in `@eggjs/typings`, not in one consumer package
- import shared global augmentations from the package entry that needs the type surface, for example `import '@eggjs/typings/global'`
- keep `declare global` files as modules by using an `import type` or `export {}`

## Testing And PR Expectations

- run the most targeted tests that validate the touched area
- include regression coverage when changing loader, cluster, agent, HTTP, or process behavior
- use Angular-style commit messages such as `fix(loader): ensure middleware order`
- keep PR descriptions clear about motivation, scope, and test evidence

## Security And Config

- review `SECURITY.md` before handling vulnerability-related work
- do not commit secrets, credentials, or local-only URLs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eggjs/egg](https://github.com/eggjs/egg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
