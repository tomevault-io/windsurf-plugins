---
trigger: always_on
description: These instructions are for Codex and other agentic coding assistants working in this repository. Treat them as the shared project contract for production-grade development.
---

# Strmify Agent Instructions

These instructions are for Codex and other agentic coding assistants working in this repository. Treat them as the shared project contract for production-grade development.

This repo uses two focused Codex TOML agent profiles:

- `.codex/agents/development.toml`: use for new features, enhancements, refactors, UI improvements, integrations, and general ongoing development.
- `.codex/agents/bug-fix.toml`: use for crashes, regressions, broken UI, playback defects, bad data handling, and production bug fixes.

MCP server configuration lives in `.codex/config.toml`. Keep this file token-free; put secrets in the user-level Codex config or environment variables.

## Project Snapshot

Strmify is an Expo Router React Native app for iOS, Android, and web. It discovers movies and series through TMDB metadata, resolves streams through Stremio-compatible add-ons, supports direct and torrent streams, integrates OpenSubtitles, and offers multiple playback paths including React Native Video, native players, TorrServer/Stremio service URLs, and a custom iOS KSPlayer bridge.

Primary stack:

- Expo 55, React 19, React Native 0.83, Expo Router typed routes.
- TypeScript with `strict: true` and `@/*` path aliases.
- MMKV-backed local storage through `utils/StorageService.ts`.
- Theme context through `context/ThemeContext.tsx` and shared themed primitives in `components/Themed.tsx`.
- Native iOS player integration through `plugins/withKSPlayer.js` and `plugins/ksplayer-bridge/`.

## Repository Map

- `app/`: Expo Router screens and route layouts. Keep routing decisions here.
- `app/(tabs)/`: Tab screens for home, search, library, and settings.
- `app/settings/`: Settings subpages for add-ons, players, subtitles, downloads, and support pages.
- `app/movie/`, `app/series/`, `app/stream/`: Media details, listing, stream selection, and playback routes.
- `components/`: Reusable UI and feature components.
- `components/coreplayer/`: Shared custom player controls, subtitle handling, stream/audio/subtitle action builders, and playback UI helpers.
- `components/nativeplayer/` and `components/ksplayer/`: Platform player wrappers.
- `clients/`: Network clients for Stremio, OpenSubtitles, TorrServer, intro data, and related services.
- `constants/`: TMDB URL constants and theme tokens.
- `context/`: App-level React context.
- `hooks/`: Shared hooks.
- `utils/`: Platform helpers, storage, library/watch history, stream parsing, subtitles, dates, and media player helpers.
- `plugins/`: Expo config plugins and native bridge source.
- `assets/`, `public/`, `sources/`, `Strmify-screenshots/`: Static assets, PWA files, SideStore source, and screenshots.

## Agentic Operating Mode

When asked to change the codebase, act autonomously unless the request is ambiguous enough that a wrong assumption would be costly.

1. Inspect the relevant files before editing. Prefer `rg` and targeted reads.
2. Identify the route, component, client, or utility boundary that owns the requested behavior.
3. Make the smallest coherent change that solves the user goal.
4. Preserve user work already present in the tree. Do not revert unrelated edits.
5. Keep the code production-grade: readable, maintainable, debuggable, and newcomer-friendly.
6. Run the most relevant non-watch verification command when practical.
7. Summarize what changed, where, and what was verified.

Ask before doing any destructive operation, dependency installation, native build, release build, or change that requires external service credentials.

## Commands

Use these commands from the repository root:

- Install dependencies: `npm install`
- Start Expo dev server: `npm start`
- Run Android dev build: `npm run android`
- Run iOS dev build: `npm run ios`
- Run web app: `npm run web`
- Type-check: `npx tsc --noEmit`
- Run Jest tests once, only when explicitly requested or clearly useful: `npx jest --runInBand`
- Run project tests in watch mode, only for interactive local work: `npm test`
- Export web build: `npm run build:web`
- Android preview build through EAS: `npm run build:android`

Tests are not required by default. Prefer `npx tsc --noEmit`, targeted builds, and clear manual verification notes unless the user asks for tests.

## Coding Standards

- Keep TypeScript strict. Avoid `any` unless the upstream API shape is genuinely dynamic; prefer small local types for TMDB, Stremio, OpenSubtitles, and player payloads.
- Use `@/` imports for project modules.
- Prefer clear names, simple control flow, and small focused helpers. New contributors should be able to trace the feature without hidden magic.
- Separate concerns: routes coordinate, components render/interact, clients call services, utilities transform data.
- Keep React hooks stable with `useCallback`, `useMemo`, and refs where they prevent real rerenders or stale closures.
- Avoid introducing global state when existing context, route params, or storage utilities fit.
- Route navigation should use Expo Router APIs and existing route shapes.
- Store persistent settings through `StorageService` and existing `StorageKeys`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vjdev1212/strmify](https://github.com/vjdev1212/strmify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
