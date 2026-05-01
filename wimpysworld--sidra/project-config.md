---
trigger: always_on
description: Minimal Apple Music desktop client. CastLabs Electron wraps `music.apple.com` directly, injecting a lightweight hook script to bridge MusicKit.js events to native platform media controls.
---

# Sidra

Minimal Apple Music desktop client. CastLabs Electron wraps `music.apple.com` directly, injecting a lightweight hook script to bridge MusicKit.js events to native platform media controls.

## Technology stack

| Component | Technology | Purpose |
|---|---|---|
| Shell | CastLabs Electron (`castlabs/electron-releases`, wvcus) | Widevine CDM for DRM playback |
| Language | TypeScript | Application code |
| Renderer | `music.apple.com` | Apple maintains the UI |
| Package manager | npm | Dependency management |
| Build | electron-builder | Platform installers (AppImage, deb, rpm, DMG, NSIS) |
| Dev environment | Nix flake + direnv | Reproducible tooling |
| Test framework | Vitest | Unit tests for src modules |

## Development commands

Run `just --list` to see all available recipes. Key entry points: `just install` → `just run` for a dev build; `just test` for unit tests; `just lint` for type-check and actionlint. Debug variants (`run-debug`, `run-devtools`, `run-inspect`) and style tooling (`measure`, `alter`) are also available.

## User-Agent

All platforms send a platform-accurate Chrome UA (`chromeUA()` in `src/main.ts`), stripping Electron identifiers that Apple Music detects and blocks. Chrome version is pinned to `144.0.0.0` to match the CastLabs ECS Chromium build.

| Platform | UA platform token |
|----------|------------------|
| macOS | `Macintosh; Intel Mac OS X 10_15_7` |
| Windows | `Windows NT 10.0; Win64; x64` |
| Linux | `X11; Linux x86_64` |

The `10_15_7` macOS version freeze is intentional - Chrome itself freezes this value to reduce fingerprinting surface.

## Conventions

- Commit messages follow [Conventional Commits](https://www.conventionalcommits.org/)
- British English spelling
- The codebase is tightly focused and as lean as possible

## Code quality objectives

**Electron security**
- `contextIsolation: true`, `nodeIntegration: false` on all windows
- All renderer→main IPC flows through the `SEND_CHANNELS` allowlist in `src/preload.ts`; blocked channels log a warning
- No Node.js APIs exposed to the renderer
- External URLs validated for `http:`/`https:` protocol before opening via `setWindowOpenHandler` and tray links

**TypeScript**
- `strict: true` in `tsconfig.json`; zero `any` annotations, `@ts-ignore`, or `@ts-expect-error` in `src/`
- IPC payloads typed via `TypedEmitter<PlayerEvents>`; no raw string channel dispatch
- CastLabs type gaps resolved via module augmentations in `src/types/electron.d.ts`, not type casts at call sites
- Hook-preload contract typed via `src/types/hook.d.ts` - declares `SidraHook`, `AMWrapperBridge`, `SendChannel`, `ReceiveChannel`, `SidraCommandMessage`, and `Window` augmentations; preload uses `Set<SendChannel>` and `Set<ReceiveChannel>` so tsc enforces channel sync at compile time

**Architecture**
- All integrations follow the `init(ctx: IntegrationContext)` pattern and manage their own lifecycle
- Platform-specific modules (`electron-updater`, MPRIS) lazy-required only when needed; never at module top level
- `playbackTimeDidChange` handlers store position only - never trigger a debounced send (see architecture notes)
- All event listeners and resources cleaned up on `will-quit`

**Tests**
- Tests cover pure logic and event forwarding; shared mock fixtures live in `test/mocks/` to avoid duplication
- Type-level assertions (`expectTypeOf`) used to verify event map contracts at compile time

**Dependencies**
- Minimise runtime dependencies; each must be purpose-driven
- Do not add dependencies that duplicate Electron or Node.js built-ins

## Internationalisation (i18n)

`src/i18n.ts` handles locale detection and translated strings for Sidra's own UI. Apple Music's web UI localises itself independently.

### Locale detection

- `app.getPreferredSystemLanguages()` returns a BCP 47 ordered list (e.g. `['en-GB', 'en']`)
- `getLoadingText()` walks the list and matches against the `LOADING_TEXT` record - exact tag first, then base language (e.g. `en-GB` → `en`)
- `getStorefront()` uses `app.getLocaleCountryCode()` to extract the region code independently of language (e.g. returns `GB` regardless of whether the language is `en`, `cy`, or `gd`), then lowercases it for use as an Apple Music storefront path segment

### Adding translations

Translation records live in `assets/locales/` as JSON files. Each file contains a map of record names to `Record<string, string>` objects keyed by BCP 47 language tags. `src/i18n.ts` loads these at startup via `fs.readFileSync` + `getAssetPath()` and re-exports all 31 named records.

| File | Records |
|------|---------|
| `assets/locales/loading.json` | `LOADING_TEXT` |
| `assets/locales/tray.json` | `ABOUT_TEXT`, `QUIT_TEXT`, `NOTIFICATIONS_TEXT`, `DISCORD_TEXT`, `START_PAGE_TEXT`, `START_PAGE_HOME_TEXT`, `START_PAGE_NEW_TEXT`, `START_PAGE_RADIO_TEXT`, `START_PAGE_ALL_PLAYLISTS_TEXT`, `START_PAGE_LAST_TEXT`, `ON_TEXT`, `OFF_TEXT`, `STYLE_TEXT`, `ZOOM_TEXT`, `PREVIOUS_TEXT`, `PLAY_TEXT`, `PAUSE_TEXT`, `NEXT_TEXT`, `VOLUME_TEXT`, `MUTE_TEXT`, `SHARE_TEXT` |
| `assets/locales/about.json` | `CLOSE_TEXT`, `VERSION_PREFIX`, `COPYRIGHT_SUFFIX`, `LICENSE_PREFIX` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wimpysworld/sidra](https://github.com/wimpysworld/sidra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-27 -->
