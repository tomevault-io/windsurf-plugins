---
trigger: always_on
description: Zaparoo App is a mobile-first React 19 + TypeScript + Vite application for Zaparoo Core. It runs on web, iOS, and Android through Capacitor 8, and talks to Core over WebSocket JSON-RPC.
---

# Zaparoo App Agent Guide

Zaparoo App is a mobile-first React 19 + TypeScript + Vite application for Zaparoo Core. It runs on web, iOS, and Android through Capacitor 8, and talks to Core over WebSocket JSON-RPC.

## Commands

`package.json` is the source of truth for scripts. Common commands:

### Development

- `npm run dev` — start the Vite dev server. Ask before running because it is long-lived.
- `npm run dev:server` — start dev mode with `NODE_ENV=development`; requires `DEV_SERVER_IP` in `.env`. Ask before running because it is long-lived.
- `npm run preview` — preview a production build locally. Ask before running because it is long-lived.

### Validation

- `npm run typecheck` — TypeScript check with `tsconfig.test.json`.
- `npm run lint` — run ESLint.
- `npm run lint:fix` — run ESLint autofixes.
- `npm run format:check` — check Prettier formatting.
- `npm run format` — apply Prettier formatting.
- `npm run test` — run Vitest.
- `npm run test:coverage` — run Vitest with coverage.

### Build, sync, and native

- `npm run build` — production build, then Capacitor sync.
- `npm run build:web` — web-only production build.
- `npm run build:core` — embedded Core build mode.
- `npm run build:server` — development server build, then Capacitor sync.
- `npm run build:analyze` — analyzer build.
- `npm run sync` — Capacitor sync only.
- `npm run live-update` — signed live update upload; requires `live-update-private.pem`.
- `npx cap open ios` / `npx cap open android` — open native projects.

## Project layout

```text
src/
  components/        React components
    ui/              shadcn/ui Radix-based primitives
    wui/             custom Zaparoo UI components
    home/            home-page components
    nfc/             NFC components
  hooks/             custom React hooks
  lib/               core utilities, stores, API, transport, crypto
  routes/            TanStack Router file-based routes
  translations/      i18next JSON files
  __tests__/         unit, integration, and validation tests
  test-utils/        render helpers, MSW handlers, factories
  __mocks__/         Capacitor plugin mocks
docs/                focused guides for Capacitor, deployment, and testing
```

## Boundaries

### Always

- Edit existing files when possible; do not create new files speculatively.
- Use the `@/` alias for imports from `src/`.
- Use Capacitor `Preferences` for persisted state; never use `localStorage`.
- Log errors with `logger.error(msg, err, { category, action, severity })`.
- Add new UI strings to `src/translations/en-US.json` only; other locales fall back.
- Keep TypeScript strict. Do not use `any`.

### Ask first

- Adding a Capacitor plugin, because it forces a native rebuild and store release.
- Bumping the app version; see `docs/deployment.md` because three files must change in lockstep.
- Refactors that touch many files or shared infrastructure.
- Force-pushes, branch deletes, force-resets, hook bypasses, or anything that overwrites others' work.
- Starting long-running local servers.

### Never

- Skip pre-commit hooks with `--no-verify` or commit secrets.
- Dismiss lint, type, or test failures as pre-existing; fix them or report the blocker.
- Mock the component under test, build fake components inside test files, or use hardcoded delays. Use `findBy*` and `waitFor` for async UI.
- Reconnect the WebSocket manually; the transport owns reconnect behavior.
- Test CSS classes; test accessible behavior instead.

## State

- Primary app state lives in `useStatusStore` at `src/lib/store.ts`.
- Persisted preferences and capability flags live in `usePreferencesStore` at `src/lib/preferencesStore.ts`.
- Wait for `_hasHydrated` before rendering UI that depends on persisted preferences.
- Read `coreVersion` from the store; do not pass it through props.
- `runQueue` is `{ value: string; unsafe: boolean } | null`. `writeQueue` is a plain `string`.
- For tests, reset stores with `useStatusStore.setState({ ... })` in `beforeEach`; there is no `getInitialState()`.

## API and transport

- `CoreAPI` in `src/lib/coreApi.ts` is the JSON-RPC client over WebSocket. Check that file for the current method inventory.
- Call `CoreAPI.reset()` between tests.
- Transport code in `src/lib/transport/` owns auto-reconnect, heartbeat, and offline queueing.
- Queues are processed by `useRunQueueProcessor` and `useWriteQueueProcessor`.

## Feature gating

- Add feature gates to `FEATURE_GATES` in `src/lib/featureGates.ts` with `since`, `marquee`, and `labelKey`.
- Add the `labelKey` to `src/translations/en-US.json` under `features`.
- Wrap gated UI in `<GatedFeature featureId="...">` or use `useCoreFeature(id)`.
- Dev versions (`-dev`, `-rc`, `-beta`, `-alpha`, or empty) pass all gates.

## Logging

- `logger.log`, `logger.debug`, and `logger.warn` are dev-only. `logger.error` always logs and reports to Rollbar in native production builds.
- Categories: `nfc`, `storage`, `purchase`, `api`, `camera`, `accelerometer`, `queue`, `connection`, `share`, `lifecycle`, `websocket`, `general`.
- Severities: `critical`, `error`, `warning`, `info`, `debug`.

## Platform and Capacitor


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZaparooProject/zaparoo-app](https://github.com/ZaparooProject/zaparoo-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
