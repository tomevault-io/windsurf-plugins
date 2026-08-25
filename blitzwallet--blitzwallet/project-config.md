---
trigger: always_on
description: Guidance for AI coding agents working in this repository. Assumes no prior knowledge of the project.
---

# AGENTS.md — BlitzWallet

Guidance for AI coding agents working in this repository. Assumes no prior knowledge of the project.

## Project Overview

BlitzWallet is a free, open-source, self-custodial Bitcoin and Lightning wallet for iOS and Android, built with React Native and Expo. Users control their own 12-word BIP39 seed phrase; there is no KYC and no custodial service. Payments run primarily on the [Spark](https://spark.info) Layer 2 network, with additional support for Lightning (via Breez Liquid SDK), Liquid, Rootstock, Boltz swaps, LNURL, Nostr Wallet Connect (NWC), stablecoins (USDB/USDT/USDC), and merchant/POS tools. Backend services (contacts, messaging, notifications, gifts, pools) use Firebase. The app is localized into 8 languages.

- License: Apache 2.0 (`LICENSE`)
- Public repo: https://github.com/BlitzWallet/BlitzWallet
- `app.json` version: `0.2.7`; releases are tagged per platform (e.g. `Android-v0.7.10-pre4`, `Spark-v0.0.7-beta`)

## Tech Stack

- **React Native 0.81.4** + **Expo ~54** (bare workflow — native `android/` and `ios/` projects are checked in; built with `react-native run-*`, not `expo run`)
- **React 19.1**, JavaScript (ES modules) with some TypeScript (`.ts`/`.tsx`, e.g. `App.tsx`); `tsconfig.json` extends `@react-native/typescript-config` but there is no `tsc` script — type-checking is not enforced in CI
- **Yarn 3.6.4** (node-modules linker, Corepack), Node >= 20
- Key SDKs: `@buildonspark/spark-sdk` (+ `spark-web-context` from a GitHub commit), `@breeztech/react-native-breez-sdk-liquid`, `@flashnet/sdk` (BTC↔stablecoin swaps), `boltz-core`, `nostr-tools`, `ethers`, `@scure/bip32`/`bip39`, `@noble/hashes`/`secp256k1`
- **Firebase** via `@react-native-firebase/*` (auth, Firestore, functions, messaging, crashlytics, storage) — see `db/` and `firebase.json`
- Navigation: `@react-navigation` v7 (native-stack, drawer, bottom-tabs, pager-view)
- State: React Context only (no Redux/Zustand) — a deeply nested provider tree in `App.tsx` backed by `context-store/*`
- UI: custom themed components + `react-native-reanimated` v4, `react-native-worklets`, `lottie`, `react-native-svg`, vision-camera, maps, webview
- i18n: `i18next` + `react-i18next`, static JSON in `locales/<lang>/translation.json`

## Repository Layout

- `App.tsx` — app entry; composes the global Context provider tree (order matters — providers consume each other's hooks) and the root `NavigationContainer`
- `index.js` — JS entry point: loads `pollyfills.js`, `disableFontScalling.js`, `i18n.js`, registers the App and `index.background.js`
- `index.background.js` — headless background-notification handler (Firebase messaging + expo-task-manager). Deliberately contains **no** App component or Context providers — keep it that way
- `app/` — application code
  - `app/functions/` — business logic, organized by domain: `spark/` (largest module: payments, transactions, restore, Flashnet swaps, spend-and-replace), `sendBitcoin/`, `receiveBitcoin/`, `boltz/`, `breezLiquid/`, `lnurl/`, `nwc/`, `contacts/`, `accounts/`, `messaging/`, `notifications/`, `pos/`, `pools/`, `savings/`, `gift/`, `lrc20/`, `webview/`, plus many single-file utilities
  - `app/screens/` — UI screens: `createAccount/` (onboarding), `inAccount/` (main app: home, send/receive, settings, POS, gifts, BTC map, analytics), plus debug screens (`boltzDebug.js`, `breezSparkTest.js`)
  - `app/components/`, `app/hooks/`, `app/constants/` (theme, styles, math, icons), `app/assets/`
- `context-store/` — ~35 React Context providers (theme, keys, auth, spark wallet, contacts, NWC, flashnet, pools, savings, POS, balances, notifications, webview, etc.)
- `navigation/` — stack/drawer/tab navigators (`GiftsStack`, `POSStack`, `PoolsStack`, `SavingsStack`, …) and `navigationService.tsx`
- `db/` — Firebase init (`initializeFirebase.js`) and Firestore access layer (`index.js`, `handleBackend.js`); user-facing data is encrypted via `app/functions/messaging/encodingAndDecodingMessages.js`
- `locales/` — translation JSON per language (`en` is the source of truth) and `localeslist.js`; contribution guide in `locales/how_to_contribute.md`
- `__tests__/` — Jest tests, mirroring source layout (`functions/`, `context-store/`, `screens/`, plus flat files)
- `patches/` — `patch-package`-style patches for native/JS deps (`@noble/*`, ecpair, pbkdf2, Lottie, LWK) — see `patches/breezSDK.md` for the manual Breez SDK Kotlin edit
- `docs/` — design docs and plans (`docs/plans/`, `docs/superpowers/`)
- `android/`, `ios/` — checked-in native projects
- `CLAUDE.md` — working-agreement rules for AI agents (see "Working Conventions" below); `CVE_AUDIT_REPORT.md` — dependency security audit (2026-07-19)
- `CLAUDE-SECURITY-*/` — artifacts of past automated security runs; not part of the app

## Build and Run Commands

All commands are Yarn scripts (`package.json`); install deps with `yarn install` (Corepack enabled). iOS additionally requires `pod install` in `ios/`.

- `yarn start` — start Metro bundler
- `yarn android` / `yarn ios` — build and run on device/emulator
- `yarn android:clean` — `./gradlew clean` then run
- `yarn lint` — ESLint over the repo (the only CI check)
- `yarn test` — Jest

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BlitzWallet/BlitzWallet](https://github.com/BlitzWallet/BlitzWallet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
