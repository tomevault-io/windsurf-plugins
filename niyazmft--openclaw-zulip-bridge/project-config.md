---
trigger: always_on
description: npm run check              # Full validation: bootstrap → typecheck → build → smoke → test → package
---

# AGENTS.md — OpenClaw Zulip Bridge

## Essential Commands

```bash
npm run check              # Full validation: bootstrap → typecheck → build → smoke → test → package
npm run typecheck          # tsc -p tsconfig.json (noEmit, type-checking only)
npm run build              # tsc -p tsconfig.build.json (emits to dist/)
npm run test               # node --test --experimental-strip-types --loader ./test-loader.js test/*.test.ts
npm run check:bootstrap    # Verifies tsc is installed (skips devDeps if NODE_ENV=production)
npm run check:smoke        # Validates built dist/ artifacts with a loader that does NOT remap .js→.ts
npm run check:package      # Validates version sync, required fields, and npm pack integrity
```

**Command order matters**: `npm run check` runs steps sequentially. Building must precede smoke tests and package checks.

## Architecture

- **Entry points**: `index.ts` (plugin) and `setup-entry.ts` (onboarding wizard). Both emit to `dist/`.
- **Core wiring**: `src/channel.ts` — the single file that glues config, accounts, messaging, security, and monitoring together via `createChatChannelPlugin`.
- **Host dependency**: `openclaw/plugin-sdk` subpaths are **not npm packages**. They are provided at runtime by the OpenClaw host. Type shims live in `types/openclaw-plugin-sdk.d.ts`; runtime test shims in `test/openclaw-plugin-sdk-shim.js`.
  - Channel plugins should import from `openclaw/plugin-sdk/channel-core` (not the legacy `openclaw/plugin-sdk/core` umbrella).
- **Monitor lifecycle**: The monitor must be started via `gateway.startAccount` inside the `base` parameter of `createChatChannelPlugin`. Putting `gateway` at the top level of the returned object causes `createChatChannelPlugin` to strip it during destructuring, resulting in the host throwing "Channel zulip does not support runtime start".
  - `gateway.startAccount(ctx)` receives `ctx.setStatus`, `ctx.abortSignal`, `ctx.account`, `ctx.accountId`, `ctx.cfg`, `ctx.runtime`, `ctx.log`.
- **Bot workspace**: `src/zulip/workspace.ts` provides sandboxed file storage under `data/zulip-workspace/{accountId}/` with path-traversal rejection, automatic cleanup, and optional Zulip upload integration.
- **Session recovery**: `src/zulip/recovery.ts` recovers interrupted messages after gateway restart. Opt-in via `enableSessionRecovery: true` (default: `false`).
- **Audit logging**: `src/zulip/audit-logger.ts` writes persistent JSON-line audit events to `{dataDir}/audit/{accountId}.audit.log` with 1MB rotation.
- **Rate limiting**: Configurable per-sender rate limit via `maxMessagesPerMinute` (default: `60`, `0` disables). Sliding 60-second window.
- **Security docs**: See `SECURITY.md` for full security policy covering credential handling, data access, and network communication.

## TypeScript Conventions

- **ESM only**: `"type": "module"` in package.json. All imports use `.js` extensions (NodeNext resolution) even though source files are `.ts`.
- **Lenient config**: `strict: false`, `noImplicitAny: false` — don't add strictness flags without asking.
- **Two tsconfigs**: `tsconfig.json` for typechecking (noEmit, includes `test/`); `tsconfig.build.json` extends it, enables emit, disables `allowImportingTsExtensions`, excludes `test/`.

## Testing

- Uses Node.js built-in test runner (`--test` flag), not Jest/Vitest.
- Custom loader (`test-loader.js`) remaps `openclaw/plugin-sdk` imports to the shim and resolves `.ts` from `.js` imports.
- Run a single test: `node --test --experimental-strip-types --loader ./test-loader.js test/policy.test.ts`
- No test fixtures or external services required.

## CI

CI runs on Node 22, uses `npm ci`, runs `npm run check:bootstrap` followed by `npm run check`, then enforces a clean working directory (`git diff --exit-code`). If check modifies any generated files, CI will fail.

## Build Artifacts

`dist/` is gitignored and must be built locally. The smoke test imports from `dist/`, so `npm run build` must succeed before `check:smoke` or `check:package` can pass.

- The **smoke test** (`scripts/smoke-test-dist.js`) is executed via `test/smoke-loader.js`, which **only** shims `openclaw/plugin-sdk` and deliberately does **not** redirect `.js` imports to `.ts`. This ensures the test exercises actual built artifacts in `dist/`, not source files.
- The **package check** (`scripts/check-package.js`) verifies version sync between `package.json` and `openclaw.plugin.json`, confirms every file in `package.json` `"files"` exists, and validates that critical artifacts and metadata are included in `npm pack --dry-run` output.

## Plugin Manifest

`openclaw.plugin.json` version must stay in sync with `package.json` version. `npm run check:package` validates this.

## Environment

Dev dependencies must be installed. `.npmrc` sets `include=dev` to prevent npm from skipping devDeps. If bootstrap fails, check that `NODE_ENV` is not set to `production`.

## Deployment

This plugin targets **any OpenClaw host** running `>=2026.6.0`. It is not limited to a specific device or platform.

### Install via ClawHub (recommended)

```bash
openclaw plugins install clawhub:@niyazmft/openclaw-zulip
```

Then restart the gateway and run `openclaw channels add` to configure.

### Manual deployment


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [niyazmft/openclaw-zulip-bridge](https://github.com/niyazmft/openclaw-zulip-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
