---
trigger: always_on
description: validates the version and publishes mutable candidate artifacts; it does not modify the PR it is
---

# Repository Guidelines

## Project Structure & Module Organization

This is a TypeScript pnpm monorepo (`packages/*`, see `pnpm-workspace.yaml`) centered on a WXT WebExtension for Twitch and Kick drop farming. Seven workspace packages:

- **`packages/extension`** — the WXT extension shell. Entrypoints are in `entrypoints/`: `background.ts` wires browser lifecycle/runtime APIs to the controller, content scripts are split by platform (`kick.content.ts`, `twitch.content.ts`, `twitchKeepAlive.content.ts`), and `popup/` mounts the React popup. Extension-specific browser adapters and helpers live in `src/core/` (`storage.ts`, `tabs.ts`, `playbackContent.ts`, `keepAliveContent.ts`, `version.ts`, `links.ts`). Tests are in `tests/**/*.test.ts`. Static assets are in `public/`. `wxt.config.ts` declares the manifest, permissions, content scripts, and localized-message copy step.
- **`packages/core`** — the browser-free farming engine imported as `@lurkloot/core`. It owns the scheduler/controller (`packages/core/src/core/scheduler.ts`, `packages/core/src/background/controller.ts`), shared tab/watch abstractions, tabless watch logic, Twitch integrity handling, and platform adapters/parsers under `packages/core/src/platforms/` (`adapter.ts`, `twitch/`, `kick/`). It must not import WXT or browser globals; `packages/extension/tests/coreBoundary.test.ts` guards this so the extension and CLI can both reuse it.
- **`packages/cli`** — the headless/Docker runtime imported as `@lurkloot/cli`. It reuses `@lurkloot/core` and `@lurkloot/shared`, provides Node transports/auth/config/storage, and builds to `dist/index.mjs`.
- **`packages/locales`** — the localized message catalog package imported as `@lurkloot/locales`. JSON catalogs live in `messages/`, and `src/index.ts` exposes the async catalog loader used by the extension and popup UI.
- **`packages/popup-ui`** — the shared React popup UI imported as `@lurkloot/popup-ui` (`Popup.tsx`, `primitives.tsx`, view components like `idleWatchlist.tsx`/`drops.tsx`/`settings.tsx`, and the rate-nudge logic), consumed by both the extension popup and the site demo.
- **`packages/shared`** — framework-agnostic shared contracts imported as `@lurkloot/shared`: `models.ts`, `settings.ts`, `messages.ts`, `categories.ts`, `i18n.ts`, `logging.ts`.
- **`packages/site`** — the Astro marketing site (deployed to Cloudflare Pages at `https://lurkloot.jamezrin.com`). Pages are in `src/pages/` (`index.astro`, `privacy.astro`, `changelog.astro`, and the SEO landing pages `twitch-drops-farmer.astro`/`kick-drops-farmer.astro`), changelog data is in `src/changelog.json` with types in `src/changelog.ts`, other content data is in `src/faq.ts`/`src/consts.ts`, and components/layouts/styles are alongside. It imports the real popup UI for the live demo. Read [docs/seo.md](docs/seo.md) before editing page metadata, structured data, or either platform landing page — the two landing pages deliberately share a shell but no copy, and must not be collapsed into one templated component.

Other top-level dirs: `docs/` (architecture and store-listing notes), `scripts/` (repo tooling), and `references/` (optional, untracked local snapshots — see below).

## Build, Test, and Development Commands

Use pnpm for all package tasks. The root `package.json` orchestrates the workspace; these scripts run from the repo root and delegate to the right package via `--filter`.

- `pnpm install`: install dependencies from `pnpm-lock.yaml`.
- `pnpm dev`: run the WXT development server for Chromium.
- `pnpm dev:firefox`: run WXT for Firefox.
- `pnpm dev:site`: run the Astro site dev server.
- `pnpm test`: run the extension Vitest suite once.
- `pnpm typecheck`: run `tsc --noEmit` across all packages (`pnpm -r typecheck`).
- `pnpm build` / `pnpm build:firefox`: create production extension builds.
- `pnpm build:site`: build the Astro site; `pnpm build:all` builds every package.
- `pnpm check`: run script tests, workspace typechecks, extension tests, and the Astro site build.
- `pnpm verify`: run `pnpm check` and both browser builds.
- `pnpm zip` / `pnpm zip:firefox`: package release artifacts into `packages/extension/.output/`.

## Cutting a Release

Label a pull request into `main` with `release/patch`, `release/minor` or `release/major`. The label
validates the version and publishes mutable candidate artifacts; it does not modify the PR it is
applied to. Merge that PR normally with a merge commit. Prepare release then cuts `release/X.Y.Z`
from the resulting merge commit on `main`, commits the version, and opens its own PR into `main`
whose diff is only the version bump. Candidate artifacts refresh on every push to a labelled head
and on every release-branch push.

Merge the generated release PR with a merge commit; **Release** starts automatically, publishes the
GitHub release, GHCR aliases, Chrome Web Store submission and production site after approval, then a
separate `sync` job — approved on `production` in its own right — merges `main` directly into
`develop` with the dedicated sync App. A hotfix is the same flow
with `release/patch` on a PR branched from `main`. Use manual **Release** dispatch only for idempotent
recovery. Do not create or move tags by hand.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jamezrin/lurkloot](https://github.com/jamezrin/lurkloot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
