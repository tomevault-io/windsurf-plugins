---
trigger: always_on
description: **Intelligo** — open-source **application framework and operational platform** for vertical AI SaaS products (NOT a starter kit, NOT another AI framework). Turborepo monorepo with pnpm workspaces (`packages/*`, `apps/*`, `tools/*`): the framework packages, the shadcn-compatible page registry, and a reference application that is the registry's canonical installed result.
---

# AGENTS.md

## Project

**Intelligo** — open-source **application framework and operational platform** for vertical AI SaaS products (NOT a starter kit, NOT another AI framework). Turborepo monorepo with pnpm workspaces (`packages/*`, `apps/*`, `tools/*`): the framework packages, the shadcn-compatible page registry, and a reference application that is the registry's canonical installed result.

This repository is the framework's home. It is edited here, and every workspace under `packages/` is released to npm as `@intelligo-dev/*` by **one release commit**: bump every published manifest to the new version and head `CHANGELOG.md` with a `## [X.Y.Z]` section. Merging it to main runs `.github/workflows/release.yml`, which builds, runs the full suite, proves a consumer can install what is about to ship — packed-package compatibility, an app scaffolded outside the monorepo from the packed tarballs that builds and boots with every registry item installed (`scripts/consumer-smoke.sh`), and `apps/app` still being exactly the CLI's output (`app:regenerate --check`) — publishes under the dist-tag the version implies (`1.0.0-beta.N` → `beta`, a plain `1.0.0` → `latest`), pushes the `vX.Y.Z` tag and creates the GitHub release from that changelog section. The publish runs after CI passes, through npm trusted publishing (OIDC, provenance, no token); deprecations in `scripts/npm-deprecations.json` are a maintainer's `node scripts/npm-maintain.mjs`, and so is the prerelease `latest` tag, which the release run only reports when it is behind (npm accepts the OIDC identity for `publish` only, so far). A version with no changelog section does not release. `apps/website` deploys intelligo.dev and serves the page registry at `/r`. Products built on the framework live in their own repositories and consume the npm packages; nothing product-specific belongs here (`tests/architecture/publishability.test.ts` enforces it).

**Decisions:** these are settled, and the maintainers keep the record of why privately — raise it with them before changing any of them: the AI-framework boundary (frameworks stay native), the composition root, the execution boundary, persistence contracts, the i18n-native registry, package topology, the headless chat transport, the design system (and its primitives' motion), the chat extension contract, and money as micros with a currency.

## The boundary (read before writing code)

- Intelligo owns SaaS infrastructure: auth, workspaces/RBAC, entitlements, credits, billing, execution/usage/cost/audit records, conversation/document/identity persistence, jobs, admin console, CLI, and the page registry.
- The developer owns the product: the AI framework used **natively** (no universal agent abstractions), prompts, tools, workflows, product data, and every installed page as **consumer-owned source**.
- **Pages ship through the registry, not through packages.** `packages/registry/` (a private workspace, never published) holds the source of the shadcn-schema items; `pnpm registry:build` emits `packages/registry/public/r/*.json`, which `apps/website` publishes at `intelligo.dev/r/<item>.json`; consumers install with the standard shadcn CLI. No runtime UI package is required to render them.
- **Installed items are used verbatim.** Product variance flows only through consumer-owned config: `lib/shell-config.tsx` (banner, header-right), `lib/nav-config.ts`, `lib/chat-config.tsx` (agent identity, starters, headerRight, auto-continue), `lib/chat-renderers.tsx`, `lib/onboarding-steps.ts`, `lib/billing-config.ts` (product slug, credit bundles), `lib/workspace-bootstrap.ts`, `lib/document-patterns.ts` — and message files. Never edit an installed component; grow a seam in `packages/registry/base/` instead.
- **Items are i18n-native.** Copy lives in per-item next-intl namespaces (`messages/en/<item>.json`, namespace = item name); page targets are `app/[locale]/...`; navigation goes through the consumer's `@/i18n/navigation`. Adding a language = adding `messages/<locale>/*.json`.
- No product vocabulary inside a framework package. A change that needs one is a missing registry or port, and that is the better pull request.
- Import-side-effect registration is banned; registries are populated from an explicit composition root. Business logic lives in package services behind ports; Server Actions and Route Handlers are thin callers.

## Commands

```bash
pnpm dev              # reference app on :4002
pnpm build            # Build all
pnpm lint / pnpm type-check
pnpm test             # Vitest (root projects config — the real suite)
pnpm vitest run path/to/file.test.ts
pnpm test:mutation    # Stryker over the scope in stryker.config.mjs (~1 min)

# Registry
pnpm registry:build   # shadcn build → packages/registry/public/r/*.json
# Install an item (run INSIDE the consumer app, absolute artifact path —
# relative paths trip shadcn 3.8's unsafe-path check on (group)/ targets):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [intelligo-dev/intelligo](https://github.com/intelligo-dev/intelligo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
