---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Wavio is a React Native / Expo music streaming client for Android (iOS WIP) that talks to multiple server types: OpenSubsonic, Navidrome and Jellyfin. It alos supports a local music library on the device's file system. Podcast features use the Taddy API. Radio stations are supported via the Radio Browser API.

## Monorepo layout

The repo is a **Bun workspace monorepo** (`workspaces: ["apps/*"]` in the root `package.json`). Two workspaces:

- `apps/mobile/` — the Expo app (everything described under **Architecture** below; all relative paths in this file are rooted here).
- `apps/landing/` — the Astro marketing website (see **Marketing site** below).

Root-level files that govern the whole repo:
- `package.json` — private root; workspace globs, cross-workspace scripts, and `patchedDependencies` (Bun applies patches at install time, so this **must** live at the root, not in `apps/mobile`).
- `bunfig.toml` — pins `linker = "hoisted"` so `node_modules` is flat; RN/Expo tooling (Metro) and `jest-expo`'s `transformIgnorePatterns` assume a hoisted layout, not Bun's isolated/symlinked default.
- `bun.lock` (single root lockfile), `.bun-version`, `patches/`, `.gitignore`.

One `bun install` at the root installs both workspaces. When bumping a patched dep (`expo-audio`, `lucide-react-native`, `zod`) keep the version exactly matching the `patchedDependencies` key — e.g. `expo-audio` is pinned to an exact `56.0.10` so its patch applies.

## Commands

Package manager is **bun** (see `bun.lock`), though README still references pnpm. Use `bun install` / `bun run <script>`.

Run from the **repo root** (delegate to a workspace via `--cwd`):
- `bun run mobile:start` / `mobile:android` / `mobile:ios` / `mobile:web` — Expo dev client (sets `DARK_MODE=media`)
- `bun run mobile:lint` / `mobile:lint:fix` — Biome check (formatter + linter, replaces ESLint/Prettier)
- `bun run mobile:test` — `jest` (preset `jest-expo`)
- `bun run mobile:typecheck` — `tsc --noEmit`
- `bun run mobile:prebuild` — regenerate `apps/mobile/android/` and `apps/mobile/ios/`
- `bun run landing:dev` / `landing:build` / `landing:preview` — Astro marketing site

Or run inside a workspace directly: `bun run --cwd apps/mobile <script>`, or `cd apps/mobile && bun run <script>`. Single mobile test: `cd apps/mobile && bunx jest __tests__/queue.store.test.ts`. APK build: `cd apps/mobile && eas build --profile preview --platform android` (profiles in `apps/mobile/eas.json`).

### Environment variables

Secrets are **not** committed to `eas.json`. They live in EAS server-side environment variables, scoped per environment (`development` / `preview` / `production`), created with `eas env:create`. Each `eas.json` build profile sets an `environment` so `eas build` (cloud **or** `--local`) pulls the matching bucket.

Where each var belongs:
- `EXPO_PUBLIC_OPENSUBSONIC_API_VERSION`, `EXPO_PUBLIC_CLIENT_NAME`, `EXPO_PUBLIC_ENV` — non-secret; kept inline in each profile's `env` block in `apps/mobile/eas.json`.
- `EXPO_PUBLIC_TADDY_PODCASTS_API_USER_ID`, `EXPO_PUBLIC_TADDY_PODCASTS_API_KEY`, `EXPO_PUBLIC_TADDY_PODCASTS_API_LANGUAGE`, `EXPO_PUBLIC_TADDY_PODCASTS_API_COUNTRY` — EAS `development` environment only (`sensitive`/`plaintext` visibility; they're `EXPO_PUBLIC_` so they end up in the bundle anyway — `secret` would be misleading). They seed the initial `stores/podcasts.ts` state (and `clearTaddyPodcastsConfig` resets back to them) so podcasts work without manually entering config; values set via the in-app settings override them and persist. Language/country must be valid `Language`/`Country` enum keys (e.g. `FRENCH`/`FRANCE`). Absent in preview/production builds → the app falls back to in-app Taddy config.
- `SENTRY_AUTH_TOKEN` — build-time only (sourcemap upload), **not** `EXPO_PUBLIC_`, so never in the bundle. EAS `preview` + `production` only. Use **`sensitive`, not `secret`**: `secret` values are not readable outside EAS servers, so a local build (`eas build --local`) can't receive them — `sensitive` can.

`EXPO_PUBLIC_*` vars are inlined into the JS bundle at build time; `EXPO_PUBLIC_` values are extractable from a shipped APK, so EAS scoping protects them in git/logs but does not make a client-embedded key truly secret.

The dev-server scripts (`start` / `android` / `ios` / `web` in `apps/mobile/package.json`) are wrapped with `eas env:exec --non-interactive development "…"` because `expo run:android` / `expo start` do **not** pull EAS env vars on their own (only `eas build` does). This injects the Taddy creds for daily dev without a `.env` file — so `.env` stays clean and secrets can't leak into a local `preview`/`production` build (`eas build --local`, which reads both `.env` and EAS). Trade-off: these scripts now require being logged into EAS and online. **Keep secrets out of `.env`**; if you do keep Taddy in a gitignored `.env` for offline convenience, strip it before any local `preview`/`production` build.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Joel-Mercier/wavio](https://github.com/Joel-Mercier/wavio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
