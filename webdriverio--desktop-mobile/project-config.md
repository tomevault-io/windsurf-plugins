---
trigger: always_on
description: AI context file for the WebdriverIO Desktop & Mobile Testing monorepo.
---

# AGENTS.md

AI context file for the WebdriverIO Desktop & Mobile Testing monorepo.

## Project Overview

This is a monorepo providing WebdriverIO services for automated testing of native desktop and mobile applications.

**Supported Frameworks:**
- **Electron** - `@wdio/electron-service` (v10.x)
- **Tauri** - `@wdio/tauri-service` (v1.x)
- **Dioxus** - `@wdio/dioxus-service` (v1.x)
- **Electrobun** - `@wdio/electrobun-service` (v0.1.x — **macOS** via CEF + **Windows** via the native WebView2 renderer (CDP), incl. multi-window/multiremote; **Linux** upstream-blocked, and deeplink/multiremote blocked on the macOS CEF path — see the package README)
- **React Native** - `@wdio/react-native-service` (v1.0.0-next.x — **Android** + **iOS**; native find/tap via Appium (UiAutomator2/XCUITest); `execute` + `mock` via Hermes CDP (debug/Metro build); full mock, deeplink, context switching, logs, parallel workers (multiremote not yet — see #446))
- **Flutter** - `@wdio/flutter-service` (v1.0.0-next.x — **Android** + **iOS**; native find/tap via appium-flutter-driver (`FLUTTER` context); `execute` (Dart expression) + `mock` (Tier-2 cooperative `wdio_flutter` Dart contract) via the Dart VM Service (debug/profile build); full mock, deeplink, context switching, logs, parallel workers (multiremote not yet — see #446)). Built on `@wdio/native-mobile-core`.

**Planned:** the generic `@wdio/mobile-service` base (ships ahead of Capacitor; React Native & Flutter converge onto it), then Capacitor and Neutralino. See [ROADMAP.md](./ROADMAP.md) for details.

## Tech Stack

| Category | Technology |
|----------|------------|
| Language | TypeScript 5.9+ (strict mode, ESM) |
| Runtime | Node.js 24 LTS |
| Package Manager | pnpm 10.27.0+ |
| Monorepo | Turborepo 2.5+ with pnpm workspaces |
| Testing | Vitest 3.2+ (unit/integration), WebdriverIO 9.0+ (E2E) |
| Linting | Biome 2.2.5 + ESLint 9.37+ |
| Build | TypeScript compiler (dual ESM/CJS) |

## Monorepo Structure

```
packages/
├── electron-service/       # Electron WDIO service
├── tauri-service/          # Tauri WDIO service
├── dioxus-service/         # Dioxus WDIO service
├── electrobun-service/     # Electrobun WDIO service
├── react-native-service/   # React Native WDIO service (Android + iOS via Appium + Hermes CDP)
├── flutter-service/        # Flutter WDIO service (Android + iOS via appium-flutter-driver + Dart VM Service)
├── native-mobile-core/     # Shared Appium-mobile layer (DeviceManager, MobileBaseLauncher, session/caps/deeplink/contexts/logs) — RN + Flutter
├── tauri-plugin/           # Tauri v2 plugin (Rust + JS)
├── dioxus-bridge/          # Dioxus bridge crate (Rust) — IPC, mocking, log forwarding
├── dioxus-embedded-driver/ # Dioxus in-process WebDriver server (Rust)
├── dioxus-driver/          # Dioxus external WebDriver proxy (Rust, Windows 'external' provider)
├── flutter-bridge/         # Flutter app-side contract (Dart, pub.dev wdio_flutter) — execute + mock + emitEvent
├── native-cdp-bridge/      # Shared CDP bridge — single + multi-target (electron, electrobun, RN)
├── native-utils/           # Cross-platform utilities
├── native-types/           # TypeScript type definitions
├── native-spy/             # Spy utilities for mocking
└── bundler/                # Build tool for packages

fixtures/
├── e2e-apps/             # E2E test applications
└── package-tests/        # Package integration test fixtures

e2e/                      # End-to-end test suites
agent-os/                 # Agent OS standards and specs
```

## Service Architecture Pattern

WDIO runs launcher and worker services in **separate processes**. Every service package splits into two classes:

```
src/
├── index.ts              # Package entry point (default=worker, named launcher=launcher)
├── launcher.ts           # Launcher service (main process)
├── service.ts            # Worker service (worker process)
├── types.ts              # TypeScript type definitions
└── constants/            # Constants and configuration
```

**Launcher** (`launcher.ts`) — runs in main process, no `browser` access:
- Hooks: `onPrepare`, `onWorkerStart`, `onWorkerEnd`, `onComplete`
- Responsibilities: binary detection, port allocation, driver spawning, capability mutation
- Throw `SevereServiceError` (from `webdriverio`) for fatal failures that should stop the runner

**Worker** (`service.ts`) — runs in worker process, receives `browser` via `before` hook:
- Hooks: `before`, `beforeTest`, `beforeCommand`, `after`, `afterSession`
- Responsibilities: API injection onto `browser`, mock lifecycle, window focus, log capture

## Logging

Use `createLogger` from `@wdio/native-utils` for all logging:

```typescript
import { createLogger } from '@wdio/native-utils';
const log = createLogger('service-name', 'module-name');
```

## Mock Architecture

Mocks span two process boundaries — an **inner mock** in the app context and an **outer mock** in the test process. The inner mock (created via `@wdio/native-spy`) intercepts real API calls inside the app. The outer mock (vitest-compatible) is used for test assertions. Call data syncs one-way from inner to outer via `update()`, serialized as JSON across CDP/WebDriver boundaries. See `agent-os/standards/global/mock-architecture.md` for details.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [webdriverio/desktop-mobile](https://github.com/webdriverio/desktop-mobile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
