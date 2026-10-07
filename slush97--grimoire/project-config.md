---
trigger: always_on
description: Desktop mod manager for Deadlock (Valve). Electron 35 + React 19 + TypeScript 5.9 + Tailwind 4, pnpm. Ships for Windows, Linux (AppImage, deb, AUR, Flatpak) and macOS (Apple Silicon only, ad-hoc signed, no auto-update: see `docs/macos.md`).
---

# Grimoire

Desktop mod manager for Deadlock (Valve). Electron 35 + React 19 + TypeScript 5.9 + Tailwind 4, pnpm. Ships for Windows, Linux (AppImage, deb, AUR, Flatpak) and macOS (Apple Silicon only, ad-hoc signed, no auto-update: see `docs/macos.md`).

Single maintainer. Part of the `grimoire-workspace` monorepo-ish root; sibling repos (`../grimoire-social`, `../vpkmerge`, ...) have their own `AGENTS.md`/`CLAUDE.md`.

## Commands

```bash
pnpm install                                     # postinstall fetches the pinned vpkmerge binary
pnpm exec electron-rebuild -f -w better-sqlite3  # rebuild the native module (after any install that touched it)
pnpm dev                                         # electron-vite with HMR
pnpm typecheck                                   # tsc -b: covers src/ AND electron/
pnpm lint
pnpm test                                        # vitest run (node env, no DOM)
pnpm i18n:check && pnpm i18n:manifest            # after touching any locale catalog
pnpm build                                       # needs GRIMOIRE_SOCIAL_BASE_URL set (any URL works locally)
pnpm package:linux | package:win | package:mac
```

Before calling a change done: `pnpm typecheck && pnpm lint && pnpm test` (plus `pnpm ui:check` for renderer UI). CI (`.github/workflows/ci.yml`) runs lint, the `ui:check` design-system ratchet, `tsc -b`, vitest, the i18n gates, then `electron-vite build`. The husky pre-push hook runs only the i18n gates.

## Architecture

Electron multi-process with context isolation on and `nodeIntegration` off.

- `electron/main/`: Node side. `ipc/*` registers `ipcMain.handle` channels; `services/*` holds the logic (file I/O, SQLite, VPK work, external APIs, archive extraction).
- `electron/preload/index.ts`: the `contextBridge` API. Pure pass-through, no logic.
- `src/`: React renderer. `pages/` (routes, HashRouter), `components/` (feature folders + `common/` primitives), `stores/` (Zustand), `lib/` (pure helpers, most of the unit tests), `types/`, `locales/`.

**Adding an IPC method:** declare it once in the `ElectronAPI` interface in `src/types/electron.ts`, add the one-line bridge in `electron/preload/index.ts` (checked with `satisfies ElectronAPI`), then the handler in `electron/main/ipc/*`. **Adding a setting:** field in `AppSettings` (`src/types/mod.ts`) plus its default in `electron/main/services/settings.ts`.

Runtime data lives in the Electron `userData` dir: `mods-cache.db` (GameBanana catalog mirror + FTS5), `stats.db` (player stats), `unknown-crc-cache.db`, `settings.json`, `mod-metadata.json`, `profiles.json`, plus asset caches.

Heavy VPK/model/texture/sound work shells out to the bundled `vpkmerge` CLI (`resources/vpkmerge/`, version + sha256 pinned in `scripts/fetch-vpkmerge.mjs`).

## Hard rules

- **No em-dashes**, anywhere: UI strings, comments, docs, commit messages. Use a colon, period or parens.
- **No telemetry.** Nothing phones home on a fresh install: no analytics, usage pings or heartbeats. Network calls only happen for features the user invoked or opted into.
- **Main process owns secrets.** API keys and the social session token (stored via `safeStorage`) never reach the renderer. The main process attaches auth headers itself.
- **Wire-format types come from `@grimoire/social-types`** (`../grimoire-social/packages/social-types/`). Never redeclare them here. Social routes are `/v1/*`, additive only; breaking changes go to `/v2/`.
- **External APIs go through a rate limiter** in `electron/main/services/rateLimiter.ts` (GameBanana, deadlock-api, Steam community, GitHub). New integrations reuse that pattern.
- **The portable profile format is Grimoire-only.** Don't claim compatibility with other mod managers in copy.
- **Visible strings are i18n keys** (see below). Never hardcode user-facing copy.

## UI and design

Read `docs/ui-conventions.md` before any renderer UI work. The short version:

- Reuse the primitives in `src/components/common/` (`ui.tsx`, `forms.tsx`, `PageComponents.tsx`, `Modal.tsx`, `menu.tsx`, `ToastStack.tsx`) before writing markup.
- Colors come from the tokens in `src/index.css` `@theme`, never raw hex or raw Tailwind palette colors.
- **The accent color is user-chosen at runtime** (`src/lib/accentColor.ts` rewrites `--color-accent*`; OLED mode swaps the surface tokens). Orange `#f97316` is only the default. Always use `accent`/`accent-hover`/`accent-foreground` utilities and never design around a specific hue.
- The z-index ladder is documented at the top of `src/index.css`. Don't invent new z values.

## i18n

`src/locales/en/translation.json` is the source of truth and the only catalog translators see (via Weblate). It must contain only keys actually referenced by `t()`/`<Tx>`. `src/locales/unwired-en.json` is an English-only to-do list of hardcoded strings still to be wired. It is never bundled or translated.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Slush97/grimoire](https://github.com/Slush97/grimoire) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
