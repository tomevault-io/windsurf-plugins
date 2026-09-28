---
trigger: always_on
description: This file is for an AI coding agent (or anyone scripting a build) that needs
---

# AGENTS.md — compiling & installing BOAR from source

This file is for an AI coding agent (or anyone scripting a build) that needs
to go from a clean checkout to a running app on a real Android device — the
exact commands, and the non-obvious constraints that make the naive path
fail. For what the app *is* and how it's designed, read `README.md` and
`ARCHITECTURE.md` first; this file only covers build/install mechanics.

## The one fact that changes everything: this is NOT an Expo Go app

`app.json`'s `plugins` includes `expo-dev-client`, and this repo has three
**custom native modules** (`modules/bundled-assets`, `modules/ram-monitor`,
`modules/voice-input`) plus `llama.rn` (native LLM inference). None of that
runs inside the generic Expo Go app from the Play Store. Every build must go
through `expo prebuild` to generate a real native Android project, then a
real native build (`expo run:android` or an EAS cloud build) — there is no
`expo start` + "scan QR code with Expo Go" path for this app. If you're
tempted to just run `npx expo start` and expect it to work standalone: it
won't — it only starts the Metro bundler, which a dev-client build (built
via one of the two paths below) then connects to.

## Prerequisites

- Node.js (any recent LTS; developed against Node 24) and npm.
- One of:
  - **A local Android SDK + NDK + JDK** (Android Studio installs all three) —
    needed for `expo run:android` (builds and installs directly on a
    connected device/emulator over USB).
  - **No local Android SDK at all** — use the EAS cloud build path instead
    (`eas.json` is already configured; needs an Expo account,
    `npx eas-cli login`, no local toolchain).
- A physical Android device with USB debugging enabled and `adb` able to see
  it (`adb devices` lists it), if installing locally rather than via EAS. The
  bounty/project this app targets explicitly requires a real device, not
  just an emulator, for final verification — but an emulator works fine for
  iterating during development.

## Compile from source and install (local Android SDK path)

```bash
git clone <repo-url>
cd aoair_app
npm install
npx expo prebuild -p android --clean   # generates ./android from app.json + plugins — gitignored, regenerate any time
npx expo run:android --device          # builds the native app AND installs it on the connected device
```

Or via the `Makefile` (`make help` lists everything):

```bash
make install        # npm install (+ tells you whether you have the Android SDK)
make run-android    # prebuild + run:android
```

**`--clean` on prebuild is not optional after touching `app.json` or its
plugins** (icon, name, any native config). Without it, `expo prebuild` can
leave a stale `android/` project with old values baked in, and a plain
`npm install` will never fix that — it never touches `android/` at all.
When in doubt, `--clean`.

`make setup` is the interactive wizard for humans (`scripts/setup.mjs`); an agent
should use the explicit commands above instead. It reads answers from stdin, so it
can be scripted if needed (e.g. `printf '2\n1\n' | node scripts/setup.mjs`).

## Release APK (no Metro needed)

```bash
npx expo prebuild -p android --clean
cd android && ./gradlew assembleRelease -PreactNativeArchitectures=arm64-v8a
# -> android/app/build/outputs/apk/release/app-release.apk
```

`plugins/withReleaseSigning.js` signs it with the key named by
`BOAR_UPLOAD_STORE_FILE`, `BOAR_UPLOAD_KEY_ALIAS`, `BOAR_UPLOAD_STORE_PASSWORD`
and `BOAR_UPLOAD_KEY_PASSWORD` in `~/.gradle/gradle.properties` (never in the
repo). Without them the release build is debug-signed, which is fine for your own
phone. An APK signed with a different key can't install over an existing BOAR:
Android requires uninstalling first, which deletes the app's downloaded models.
A first release build takes about 40 minutes.

## No local Android SDK: build via EAS instead

```bash
npm install
npx eas-cli login                                      # one-time, needs an Expo account
npx eas-cli build --platform android --profile preview # builds an installable .apk in the cloud
```

or `make build-eas`. This produces a downloadable `.apk` (see `eas.json`'s
`preview` profile) — download it and `adb install <file>.apk`, or transfer it
to the device directly.

## After install: the app is not immediately usable — one more step

First launch shows a **mandatory, one-time setup screen** that downloads the
default model (Qwen2.5-1.5B + an embedding model, about 1 GB total) — this is the app's only required network access, and the app is
gated behind it (`ModelManager.requiredModelsPresent()` in
`src/models/ModelManager.ts` decides whether the chat screen or the setup
wizard shows). If you're scripting an unattended install-and-verify flow,
this download has to complete (or you pre-seed the files — see next
section) before the app is otherwise usable. After that first setup, the
app works fully offline.

### Skipping the in-app download (pre-seeding models)

If you already have the GGUF weight files on the machine running the build,
you can bake them into the APK itself instead of downloading them at
first-run — see `docs/MODELS.md`'s "Pre-seeding models you already have
locally" section for the exact filenames/checksums and the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rferrari/boar-app](https://github.com/rferrari/boar-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
