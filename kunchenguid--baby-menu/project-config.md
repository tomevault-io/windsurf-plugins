---
trigger: always_on
description: This file provides guidance for developing baby-menu itself.
---

# AGENTS.md

This file provides guidance for developing baby-menu itself.
Embedded agents launched from baby-menu should work from the active extension workspace and follow the copied `AGENTS.md` there for extension authoring.

## Commands

- `pnpm dev` - runs `scripts/dev.mjs`, prepares a gitignored `extensions-dev/` workspace by copying `extensions/AGENTS.md`, `extensions/babymenu-env.d.ts`, and `extensions/recipes/`, builds bundled ACP adapters into `out/adapters/`, and runs `electron-vite dev` from the current checkout. The app itself sees current uncommitted changes, while the embedded agent is launched inside `extensions-dev/`.
- `pnpm dev:reset` - removes `extensions-dev/` and `.cache/baby-menu/acp-sessions`, recreates the dev workspace with the latest `extensions/AGENTS.md`, `extensions/babymenu-env.d.ts`, and `extensions/recipes/`, and starts dev mode.
- `pnpm build` - build main, preload, renderer, and bundled ACP adapter bundles into `out/`.
- `pnpm generate:contracts` - regenerates `extensions/babymenu-env.d.ts` (the `@babymenu/contracts` surface) from `src/shared/contracts.ts`. Run after changing any extension-facing type or `src/shared/extension-contract-names.ts`, then commit the result; CI fails on a stale file.
- `pnpm package:mac` - cleans `release/`, builds the app, packages `release/mac-universal/Baby Menu Dev.app`, and ad-hoc signs it for local testing. Local/dev packaging uses `electron-builder.dev.yml`, which overrides `appId` to `com.kunchenguid.baby-menu.dev` and `productName` to `Baby Menu Dev` so locally-built bundles never collide with the released app (`com.kunchenguid.baby-menu`) in macOS LaunchServices. The CI release workflow builds from `electron-builder.yml` directly and keeps the production identity.
- `pnpm dist:mac` - runs `package:mac` and creates `release/Baby-Menu-<version>-universal.dmg` from the dev bundle.
- `pnpm test` - run all Vitest tests.
- `pnpm test:e2e` - run only `tests/e2e-*.test.ts` (these include real `acpx/runtime` coverage against `acp-mock` plus bundled adapter coverage against fake local CLIs).
- `pnpm typecheck` / `pnpm lint` - both run `tsc --noEmit` against `tsconfig.json`.
- Single test: `pnpm vitest run tests/<name>.test.ts` (or `pnpm vitest run -t "<name pattern>"`).

Use `pnpm` (declared `packageManager: pnpm@11.1.1`). Renderer dev server is pinned to port 5273 (`strictPort: true`).

### Packaging hygiene for automation (no-mistakes and other agents)

Agents running in `no-mistakes` worktrees (or any throwaway checkout) must not leave packaged macOS bundles behind.
A `release/mac-universal/*.app` left on disk gets auto-registered by macOS LaunchServices, and stale registrations make `open -a "Baby Menu"`, login items, and bundle-id launches resolve to the wrong build.
Follow these rules:

- If a packaged bundle is genuinely required, build it (it will carry the `com.kunchenguid.baby-menu.dev` identity), then delete the entire `release/` directory before the run finishes so nothing is left for LaunchServices to register.
- Never set a locally-built bundle as a macOS login item, and never install one into `/Applications`. The released app is delivered only through the Homebrew cask.

## Dev mode helpers

- `BABY_MENU_KEEP_POPOVER_OPEN=1` disables the blur-to-hide behavior so the popover stays open while devtools / external windows have focus.
- `BABY_MENU_AGENT=<agent-name>` overrides agent auto-detection when no saved Settings choice exists. E2E tests pass `acpx-mock` via `registryOverrides`.
- `BABY_MENU_AGENT_TIMEOUT_MS=<ms>` overrides the embedded-agent request timeout.
- `BABY_MENU_TELEMETRY=0` (or `false` / `off`) disables packaged-release telemetry; `BABY_MENU_UMAMI_HOST` and `BABY_MENU_UMAMI_WEBSITE_ID` override the self-hosted Umami target for telemetry testing.
- `process.env.VITEST` is checked in `src/main/app.ts` so importing the main entry from tests does not auto-start the Electron app.

## Architecture

This is a macOS tray-bar Electron app whose distinguishing idea is that an embedded agent (running via `acpx/runtime`) edits the active extension workspace at runtime.
Tracked source extensions use git as the accept/rollback mechanism when selected explicitly; packaged mode edits `~/.baby-menu/extensions` and uses filesystem snapshots.

Three processes, kept deliberately separate:

1. **Main** (`src/main/`) - app lifecycle, tray, popover window, IPC, git, agent runtime. Never call agent or git from the renderer directly.
2. **Preload** (`src/preload/index.ts`) - the stable bridge. Exposes `window.babyMenu` via `contextBridge`. Do not add one-off preload methods for each widget.
3. **Renderer** (`src/renderer/`) - React UI: `AgentChat`, `WidgetHost`, `SettingsView`, `UpdateIndicator`, and app-shell controls such as Quit. Widgets and extension settings sections should be hot reloadable and should not require an Electron restart for each new capability. The app shell and extension renderer surfaces share one design system, `@babymenu/ui` (`src/ui/`); see "Design system" below.
4. **Extension server actions and background tasks** - privileged filesystem, shell, network, credential, token, storage, notification, and background work should live behind extension-owned `server.ts` modules.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kunchenguid/baby-menu](https://github.com/kunchenguid/baby-menu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
