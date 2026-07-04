---
trigger: always_on
description: Monorepo using npm workspaces with two workspace roots:
---

# Contributing to vitest-mobile

## Project Structure

Monorepo using npm workspaces with two workspace roots:

- `packages/vitest-mobile/` — the main package (Vitest custom pool + runtime + native modules + CLI)
- `test-packages/` — example test modules (counter, greeting, toggle, todo-list)

Root-level files (`index.js`, `index.ios.js`, `vitest.config.ts`) are auto-generated harness entry points.

```
vitest-mobile/
├── packages/
│   └── vitest-mobile/           # The main package
│       ├── src/
│       │   ├── node/            # Vitest plugin, pool worker, device control
│       │   ├── runtime/         # Device-side: runner, render, locators
│       │   ├── babel/           # Test file wrapper plugin
│       │   ├── metro/           # Metro config helpers + test registry generator
│       │   └── cli/             # CLI commands (boot, build, debug, screenshot)
│       ├── ios/                 # Native TurboModule (Objective-C++)
│       ├── android/             # Native TurboModule (Java/JNI)
│       ├── dist/                # Built output (tsup)
│       └── tests/               # Unit + integration + e2e tests
├── test-packages/               # Example test modules
├── vitest.config.ts             # Root Vitest config (ios + android projects)
├── index.js / index.ios.js      # Auto-generated harness entry points
└── .github/workflows/ci.yml     # CI pipeline
```

## Architecture Overview

Test files are transformed by a Babel plugin (injected automatically via a custom Metro transformer) that wraps `describe()`/`it()` calls in an `exports.__run` function, making them safe to `require()` without an active runner context. The runner calls `__run()` inside `startTests()` where vitest's suite collector is active.

For a full architecture walkthrough, see [`packages/vitest-mobile/docs/architecture.md`](packages/vitest-mobile/docs/architecture.md).

## Prerequisites

| Tool         | Version      | Notes                                             |
| ------------ | ------------ | ------------------------------------------------- |
| Node.js      | >= 18        | LTS recommended                                   |
| npm          | >= 9         | Ships with Node 18+                               |
| Xcode        | >= 15        | iOS only — includes `xcrun simctl`                |
| Android SDK  | API 35       | Android only — includes `adb`, `avdmanager`       |
| Java         | 17 (Temurin) | Android only                                      |
| Vitest       | ^4.0         | Peer dependency                                   |
| React Native | >= 0.81.5    | New Architecture (Fabric + TurboModules) required |

## Getting Started

```bash
git clone <repo-url>
cd vitest-mobile
npm install
npm run build
```

## Development Workflow

### Building the Package

```bash
# One-time build
npm run build

# Watch mode (rebuilds on source changes in packages/vitest-mobile/src)
npm run dev
```

The dev loop:

1. Make code change in `packages/vitest-mobile/src/`
2. tsup watch (`npm run dev`) rebuilds `dist/`
3. Metro detects change in `dist/` and serves updated bundle
4. App reloads (may need manual relaunch — see Common Issues below)
5. Verify via screenshot + CDP eval + log tailing

### Running Tests Locally

```bash
# Boot a device
npx vitest-mobile boot-device --platform ios

# Build + install the test harness app (~5 min first build, cached after)
npx vitest-mobile bootstrap --platform ios

# Run all tests
npx vitest run --project ios

# Watch mode (re-runs on file changes)
npx vitest --project ios
```

Replace `--platform ios` with `--platform android` for Android. Android also supports `--headless --api-level 35`.

### Iterating on Components

1. Write a component + test with `pause()` at the point you want to inspect
2. Run the test via `npx vitest --project ios`
3. Test executes up to `pause()` and blocks
4. Take a screenshot: `npx vitest-mobile screenshot --platform ios`
5. Edit the component — Metro HMR updates it live on the device
6. When satisfied, remove `pause()` and the test runs to completion

### Code Quality

```bash
npm run lint          # ESLint
npm run check-types   # TypeScript
npm run format        # Prettier (write)
npm run format:check  # Prettier (check only)
```

All four must pass before merging — CI enforces this.

## CLI Commands

All commands: `npx vitest-mobile <command>`

### Device & App Lifecycle

```bash
npx vitest-mobile boot-device --platform ios
npx vitest-mobile build --platform ios
npx vitest-mobile install --platform ios
npx vitest-mobile bootstrap --platform ios        # build + install in one step

# Manual launch on simulator
xcrun simctl terminate booted com.vitest.mobile.harness
xcrun simctl launch booted com.vitest.mobile.harness --initialUrl "http://127.0.0.1:8081"
```

In a TTY, `--platform` can be omitted on most commands and you'll be prompted to pick one. In CI / non-TTY contexts, omitting `--platform` errors for commands that can't sensibly default to "both" (build, bootstrap, boot-device, reset-device). Fast filesystem-only commands (`trim-cache`, `clean-devices`, `bundle`) default to both platforms when `--platform` is omitted.

### Debugging & Inspection

```bash
npx vitest-mobile debug eval "<expression>"
npx vitest-mobile debug open

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [phantom/vitest-mobile](https://github.com/phantom/vitest-mobile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
