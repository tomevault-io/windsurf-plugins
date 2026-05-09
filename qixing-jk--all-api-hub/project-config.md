---
trigger: always_on
description: - `src/entrypoints/`: WXT extension entrypoints for `background`, `content`, `popup`, `options`, and `sidepanel`.
---

# Repository Guidelines

## Project Structure & Module Organization

### Source Modules

- `src/entrypoints/`: WXT extension entrypoints for `background`, `content`, `popup`, `options`, and `sidepanel`.
- `src/features/`: feature-oriented UI modules; keep entrypoints thin and push reusable logic into features, services, hooks, or utils.
- `src/components/` and `src/components/ui/`: shared React components and UI primitives.
- `src/services/`: business logic, persistence, site adapters, and browser integration.
- `src/hooks/`, `src/contexts/`, `src/utils/`, `src/types/`, `src/constants/`, `src/lib/`, `src/styles/`: shared app building blocks.
- `src/locales/`: app i18n resources; manifest strings live in `src/public/_locales/`.

### Tests and Artifacts

- `tests/`: Vitest setup, MSW handlers, and shared test utilities.
- `e2e/`: Playwright end-to-end coverage.
- Build artifacts are written to `.output/`; browser test artifacts may appear in `coverage/`, `playwright-report/`, and `test-results/`.

## Domain Knowledge: Site Types & Upstream Backends

This repo's `siteType` values are compatibility buckets used by `src/services/apiService/*` and related UI routing.

When working on a site type:

1. Confirm current in-repo behavior first in `src/constants/siteType.ts`, `src/services/siteDetection/detectSiteType.ts`, and `src/services/apiService/index.ts`.
2. Verify upstream behavior before making definitive claims when backend differences matter.
3. If upstream behavior cannot be verified, state assumptions clearly and ask for the target deployment URL, fork, version, or a redacted network trace.

### Relationships

- **One API (`one-api`)** is the original upstream family; many compatible deployments use `src/services/apiService/common/`.
- **New API (`new-api`)** is downstream of One API and mainly uses `src/services/apiService/common/`.
- **Veloera (`Veloera`)** and several other supported variants are downstream of New API; Veloera keeps dedicated overrides in `src/services/apiService/veloera/`.
- **OneHub (`one-hub`)** is downstream of One API with a substantially different surface.
- **DoneHub (`done-hub`)** is downstream of OneHub and currently layers `src/services/apiService/doneHub/` on top of `src/services/apiService/oneHub/` in `src/services/apiService/index.ts`.
- **Octopus (`octopus`)** has dedicated managed-site logic and API overrides in `src/services/apiService/octopus/` plus related provider logic under `src/services/managedSites/providers/`.
- **AxonHub (`axonhub`)** is not One-API/New-API compatible; it uses dedicated GraphQL admin integration in `src/services/apiService/axonHub/` plus a managed-site provider in `src/services/managedSites/providers/axonHub.ts`.
- **Claude Code Hub (`claude-code-hub`)** is not One-API/New-API compatible; it uses dedicated admin/provider integration in `src/services/apiService/claudeCodeHub/` plus a managed-site provider in `src/services/managedSites/providers/claudeCodeHub.ts`.
- **AnyRouter (`anyrouter`)** and **WONG公益站 (`wong-gongyi`)** have custom check-in handling.
- **Sub2API (`sub2api`)** is not One-API/New-API compatible; it has a different auth model and API surface.

### Managed Sites

`ManagedSiteType` is defined in `src/constants/siteType.ts` and currently includes:

- `new-api`
- `Veloera`
- `done-hub`
- `octopus`
- `axonhub`
- `claude-code-hub`

Do not assume `one-hub` or every New-API-like deployment is a managed site without checking the current type definition.

### Backend Notes

- Shared One-API/New-API-family helpers live in `src/services/apiService/common/`.
- Compatible user-id headers are handled in `src/services/apiService/common/utils.ts` and related helpers.
- AxonHub keeps its own admin integration under `src/services/apiService/axonHub/` and managed-site provider logic under `src/services/managedSites/providers/axonHub.ts`.
- Claude Code Hub keeps its own admin/provider integration under `src/services/apiService/claudeCodeHub/` and managed-site provider logic under `src/services/managedSites/providers/claudeCodeHub.ts`.
- Some adapter directories under `src/services/apiService/` are provider-specific integrations rather than `siteType` values, so check `src/constants/siteType.ts` before documenting behavior.

### Default Upstream References

When the user names a backend without a deployment URL or fork, treat these as the default upstream references:

- One API: `https://github.com/songquanpeng/one-api`
- New API: `https://github.com/QuantumNous/new-api`
- Veloera: `https://github.com/Veloera/Veloera`
- OneHub: `https://github.com/MartialBE/one-hub`
- DoneHub: `https://github.com/deanxv/done-hub`
- AxonHub: `https://github.com/looplj/axonhub`
- Claude Code Hub: `https://github.com/ding113/claude-code-hub`
- Sub2API: `https://github.com/Wei-Shaw/sub2api`

If the user's reported behavior differs from upstream, ask for the exact deployment, fork, or version before concluding the repo is wrong.

## Build, Test, and Development Commands

### Prerequisites

Node.js version from `.nvmrc` and pnpm 10+.

### Local Development

- Install: `pnpm install` (runs `wxt prepare` via `postinstall`).
- Dev, Chromium: `pnpm dev`, then load `.output/chrome-mv3-dev` as an unpacked extension.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qixing-jk/all-api-hub](https://github.com/qixing-jk/all-api-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
