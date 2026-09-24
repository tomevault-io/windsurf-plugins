---
trigger: always_on
description: Farm.js is a full-stack framework for product-integrated applications. It combines Vite-powered development with typed app-directory routing, React Server Components and streaming SSR, server actions and queries, first-party integrations, multiple renderers, and production output through Nitro-backed deployment targets.
---

# Farm.js

Farm.js is a full-stack framework for product-integrated applications. It combines Vite-powered development with typed app-directory routing, React Server Components and streaming SSR, server actions and queries, first-party integrations, multiple renderers, and production output through Nitro-backed deployment targets.

Treat the checked-out source, package manifests, tests, and current examples as the source of truth. The framework is moving quickly; do not reconstruct an API from memory, an older beta, another framework, or a stale prose document. Read `skills/farmjs/SKILL.md` before changing public framework behavior, configuration, integrations, renderers, templates, or documentation.

## Product contract

Farm should feel like one coherent framework rather than a collection of unrelated features. Preserve these promises:

- **Fast feedback.** Development startup, transforms, HMR, and navigation should stay lightweight. Do not move production-only work onto the ordinary development path.
- **Predictable full-stack boundaries.** Server code and secrets stay off the client. Client bindings, generated types, and route behavior must agree with the server implementation.
- **Typed conventions.** File routes, configuration, APIs, integrations, queries, actions, and generated declarations should fail clearly at build or type-check time when possible.
- **Safe production output.** A feature is not complete when it works only in Vite development. Verify the affected production runtime and deployment contract.
- **Progressive capability.** Experimental or optional functionality is opt-in, has a safe fallback, and adds no runtime or bundle cost when disabled where practical.
- **Renderer choice.** React is the default, but framework-level behavior should remain renderer-neutral unless the feature is explicitly renderer-specific.
- **Web-platform semantics.** Preserve native request, response, URL, cookie, form, stream, and browser behavior. Avoid surprising framework-only substitutes when the platform already defines the contract.

## Maintainer taste

Prefer the smallest change that fixes the underlying mechanism. Reuse an existing routing, rendering, integration, plugin, storage, observability, or deployment primitive before creating a parallel abstraction.

Do not add machinery merely because it looks architecturally complete. Understand the real constraint, preserve supported behavior, and make the correct path obvious. Keep hot paths direct, public APIs small, and compatibility code isolated. A new flag is not a substitute for fixing a bad default.

Follow established repository patterns unless the task exposes a concrete reason they cannot work. Explain any intentional divergence in the pull request.

## Shared language

- **app** means a project built with Farm.
- **framework** or **core** means `@farm.js/core`, including routing, build, server, client, and deployment contracts.
- **renderer** means the React, Preact, Solid, Vue, or Svelte adapter and its client/server/Vite bindings.
- **integration** means a typed provider capability mounted through Farm's integration model.
- **provider** means the external SDK or service wrapped by an integration.
- **target** means a first-class Farm deployment target; **preset** means direct Nitro preset pass-through.
- **template** means a project emitted by `@farm.js/create-app`; **example** is a maintained runnable application in `examples/`.
- **dev path** means Vite-backed development behavior; **production path** means built Nitro/runtime behavior.

## Where code lives

- `packages/farm` - `@farm.js/core`: configuration, routing, build pipeline, Vite and Nitro bridges, server rendering, client navigation, APIs, middleware, storage, docs runtime, and shared framework types.
- `packages/farm-cli` - `@farm.js/cli`: user-facing commands. Keep orchestration here and framework behavior in core.
- `packages/create-farm-app` - project generator, templates, renderer overlays, and integration starters.
- `packages/farm-react`, `farm-preact`, `farm-solid`, `farm-vue`, `farm-svelte` - renderer packages. Shared renderer expectations belong in `packages/farm-renderer-tests`.
- `packages/farm-integration-utils` - provider-neutral integration helpers. Dedicated `packages/farm-*` integration packages are the canonical authoring surface.
- `packages/farm-integrations` - compatibility re-exports for older `@farm.js/integrations/*` imports; do not use it as the implementation home for new integrations.
- `packages/farmjs-plugin` - official framework plugin package.
- `docs` - the Farm documentation application and generated route declarations.
- `examples` - maintained end-to-end usage and deployment fixtures. Prefer the closest current example over invented usage.
- `tests` and root Playwright configs - cross-package and production browser coverage.
- `scripts` - workspace test, generated-artifact, release, and publishing workflows.

## Architecture and public boundaries


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [farming-labs/farm.js](https://github.com/farming-labs/farm.js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
