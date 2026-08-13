---
trigger: always_on
description: This is `cloudflare-tools`, a monorepo with tools for developing Cloudflare Workers.
---

This is `cloudflare-tools`, a monorepo with tools for developing Cloudflare Workers.

## Overview

- Package manager: `bun`
- Testing framework: `vitest` (+ `@effect/vitest` in packages that use Effect)
- Linter: `oxlint`
- Formatter: `oxfmt`

Code must pass linting, formatting, and typechecking. Use `bun run check` to run all checks.

## Packages

### Published Packages

- `packages/astro`: Programmatic Astro integration implementing the framework-core `Framework` service, with the deploy target passed as a value; the wrangler-free Cloudflare target (a fork of `@astrojs/cloudflare` over `cloudflare-vite-plugin`) ships at the `./cloudflare` subpath.
- `packages/cloudflare-rolldown-plugin`: Rolldown plugin for Cloudflare Workers.
- `packages/cloudflare-runtime`: Effect-native local runtime for Cloudflare Workers, powered by `workerd`.
- `packages/cloudflare-vite-plugin`: Vite plugin for Cloudflare Workers; composes `cloudflare-rolldown-plugin` and `cloudflare-runtime`.
- `packages/framework-core`: Platform-neutral framework-integration core: the `BuildOutput` contract, the `alchemy:build-output` Vite collector plugin, the project module loader, the `Framework` service contract, and the `DeployTarget` contract (the deploy target as a value passed to framework integrations).
- `packages/nuxt`: Wrangler-free Nuxt integration implementing the `Framework` service (programmatic build over the project's `@nuxt/kit`; the Cloudflare target — nitro's `cloudflare_module` preset with `deployConfig` off and the entry/exports seam — ships at `@distilled.cloud/nuxt/cloudflare`).
- `packages/octane`: Wrangler-free OctaneJS integration implementing the `Framework` service. Octane wraps Vite, so the integration is thin: it drives the project's own `vite build` (whose Octane plugin builds client + server and whose `@octanejs/adapter-cloudflare` emits `dist/server/worker.js`) and maps the on-disk output onto `BuildOutput` — no adapter forks, no plugin injection. The Cloudflare target (adapter-contract constants) ships at `@distilled.cloud/octane/cloudflare`.
- `packages/sveltekit`: Wrangler-free SvelteKit integration implementing the `Framework` service; the Cloudflare deploy target (in-memory kit adapter + rolldown re-bundle for workerd + dev stub platform) ships at `@distilled.cloud/sveltekit/cloudflare`.
- `packages/waku`: Wrangler-free Waku integration implementing the `Framework` service (platform-neutral programmatic build/dev; the Cloudflare target — `cloudflare-vite-plugin` injection + the adapter fork — ships at `@distilled.cloud/waku/cloudflare`).

### Internals

- `upstream/workers-sdk/*`: Git submodule containing the `cloudflare/workers-sdk` repository.
- `fixtures/*`: Framework fixtures driven by the e2e harness (`e2e dev/build/preview` + Playwright smoke tests in both `live` and `dev` modes).
- `packages/tools/*`: Internal build and test utilities, including `packages/tools/e2e` (the fixture harness; target-scoped config carriage in `e2e.config.ts`).
- `packages/vendor/*`: Vendored-in packages from `cloudflare/workers-sdk`. See `packages/vendor/README.md` for more details.

## Framework Integrations

Framework packages (waku/astro/sveltekit, later nextjs) separate two concerns:

- The **framework half** — programmatic build/dev orchestration implementing framework-core's `Framework` service (`{ build, dev }` returning the `BuildOutput` contract).
- The **deploy-target half** — everything platform-specific (adapter/integration forks, bundler plugin injection, finishing passes, preview serving), passed to the framework as a `DeployTarget` value (a prop). Cloudflare is the first target; each framework ships its implementation as a subpath module (e.g. `@distilled.cloud/waku/cloudflare`). Future platforms (AWS) implement the same seams without touching framework packages or framework-core.

The precise `DeployTarget` contract, the harness config carriage, and the per-framework migration recipes live in `packages/framework-core/README.md` — read that before touching any framework package.

Doctrine for all framework/target work:

- **Wrangler-free, programmatic-only.** No `wrangler.json` is read or written anywhere; all worker configuration is in-memory options (plugin/adapter options here; Worker props on the alchemy side). We never spawn a framework's CLI binary (upstream pipelines may internally — that is upstream orchestration, not ours).
- **Platform-proxy policy.** Wherever an upstream integration uses wrangler's `getPlatformProxy` (SvelteKit `adapter-cloudflare`, OpenNext `initOpenNextCloudflareForDev`, Astro `platformProxy`), reimplement the feature in `@distilled.cloud/cloudflare-runtime` (workerd-backed Node-side proxies for `env`/`cf`/`ctx`/`caches`, configured in-memory) — never take a wrangler dependency.
- **Version pinning.** The upstream surfaces these integrations touch are `@experimental`/`unstable_`/unexported: pin exact framework versions, e2e-test against real apps in CI, and treat version bumps as deliberate migrations, not routine updates.

## File Conventions

In the `cloudflare-runtime` package:

- `.worker.ts` files represent internal Cloudflare Workers, checked against `@cloudflare/workers-types`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alchemy-run/cloudflare-tools](https://github.com/alchemy-run/cloudflare-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
