---
trigger: always_on
description: This file applies to `RunanywhereAI/runanywhere-android`, a standalone Android app that
---

# Android RunAnywhereAI example

This file applies to `RunanywhereAI/runanywhere-android`, a standalone Android app that
consumes the RunAnywhere Kotlin SDK from Maven Central. Run commands from the repository
root unless noted otherwise.

Single Gradle module, `:app`. minSdk 24, compile and target SDK 37, Java 17, AGP 9.2.1,
Gradle 9.6.0. The Gradle daemon needs JDK 21 (`gradle/gradle-daemon-jvm.properties` pins
`toolchainVersion=21`) even though the app compiles against 17.

## Common commands

```bash
./scripts/smoke.sh                         # Fast static SDK-usage check
./scripts/verify.sh                        # Strict debug APK build gate
./gradlew :app:assembleDebug               # Debug APK
./gradlew :app:testDebugUnitTest           # JVM tests
./gradlew :app:lintRelease                 # Release lint
```

Instrumented tests compile against the debug variant by default; pass
`-Prunanywhere.testBuildType=release` to run them against the minified, signed release
variant instead.

## SDK dependency

The app consumes the RunAnywhere SDK entirely from Maven Central. There are no local AARs
and no relative paths into any SDK source tree. Coordinates live in
`gradle/libs.versions.toml` under the single `runanywhere` version:

| Coordinate | Role |
|---|---|
| `io.github.sanchitmonga22:runanywhere-sdk` | Core SDK and the commons native library |
| `io.github.sanchitmonga22:runanywhere-llamacpp` | llama.cpp backend (LLM, VLM) |
| `io.github.sanchitmonga22:runanywhere-onnx` | ONNX Runtime (embeddings) and Sherpa-ONNX (STT, TTS, VAD) in one AAR |
| `io.github.sanchitmonga22:runanywhere-qhexrt-android` | QHexRT backend (Qualcomm Hexagon NPU), arm64 only |

The four move in lockstep; never mix versions across them. To move to a new SDK release,
bump `runanywhere` in `gradle/libs.versions.toml`, then regenerate both reproducibility
files. The README's "SDK dependency" section has the exact procedure and its two traps:
regenerate the checksums against a throwaway `GRADLE_USER_HOME` (a warm cache silently
omits parent POMs and BOM metadata), and keep both the `-linux.jar` and `-osx.jar`
`com.android.tools.build:aapt2` entries so the one committed file satisfies CI and macOS
developers alike.

Dependency verification and strict dependency locking are enforced in CI with no bypass
flags. If a bump breaks them, regenerate the files; never re-add
`--dependency-verification=off` or `env -u CI` to `.github/workflows/ci.yml`.

For a monorepo change that is not yet on Maven Central, publish it with
`publishToMavenLocal` and build with `-Prunanywhere.useLocalSdkAars=true
--dependency-verification=lenient`. Both flags are per-invocation only; never commit
either, and never set them in CI.

The published POMs supply the SDK's own transitive dependencies (wire-runtime, okhttp,
coroutines-core, okio, kotlin-stdlib, kotlinx-serialization-json, androidx core-ktx), so
`app/build.gradle.kts` declares only what the app itself uses. The one exception is
`kotlinx-coroutines-android`, declared directly so the Android artifact does not skew
below the `-core` version the SDK POM pins.

## Build configuration

`app/build.gradle.kts` reads backend configuration from environment variables first, then
the gitignored `local.properties`. `RUNANYWHERE_BASE_URL` and `RUNANYWHERE_API_KEY` must
be both set or both blank; the configuration phase fails otherwise. Blank means the SDK
initializes in its development environment, which is the normal open-source path.

ABI filters differ per variant on purpose: release ships arm64-v8a only (QHexRT is
arm64-only hardware and a single slice roughly halves the APK), debug adds x86_64 so
emulator development still works with CPU backends.

`bundleRelease` and `signReleaseBundle` depend on `verifyPlayRelease`, which checks real
upload signing, an HTTPS control-plane URL, and the upload certificate SHA-256 against
`UPLOAD_CERT_SHA256`. `bundleRelease` also depends on `generateReleaseSbom`.

## Production release requirements

A production build needs a real org-scoped API key and backend base URL — the same pair
used by `runanywhere-ios`'s `RunAnywhereLocalSecrets.plist`, `runanywhere-electron`'s
`.env`, and `runanywhere-web`'s Vercel production env. Set them in the gitignored
`local.properties` as `runanywhere.apiKey` / `runanywhere.baseUrl`; ask a maintainer for
current production credentials. Never hardcode them in any committed file.

A production build must resolve the SDK only from Maven Central — never with
`-Prunanywhere.useLocalSdkAars=true`, which is for local monorepo iteration only and must
never be used for a release artifact. Before testing a "production" build, confirm no
local Gradle property or env var is forcing the local-AAR path.

Emulator/CI passing is not sufficient. Smoke-test on real hardware: install the built APK
on a connected physical device (`adb devices`, `adb install -r ...` or
`./gradlew :app:installDebug`) and confirm cold start, model-catalog population, and at
least one real model load/inference — ideally one exercising the QHexRT/Hexagon NPU
backend, since that is this app's differentiator and cannot be validated on x86 emulators.

A signed release AAB additionally needs the real Play upload keystore

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RunanywhereAI/runanywhere-android](https://github.com/RunanywhereAI/runanywhere-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
