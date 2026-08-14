---
trigger: always_on
description: Guidance for Claude Code (claude.ai/code) when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) when working in this repository.

## Project

B-SideLoader is an Android app (Kotlin + Jetpack Compose) that finds, installs and auto-updates
APKs published on **GitHub releases** and in **Telegram channels** — an Obtainium-like app store.
It can also install a local APK. Targets Android 8.0+ (`minSdk 26`), `compileSdk`/`targetSdk` 37.
More sources may be added later; the architecture is built for that (see *Adding a source*).

## Build & run

Gradle wrapper (`./gradlew` / `gradlew.bat`), version catalog at `gradle/libs.versions.toml` — add
or bump dependencies **there**, referenced as `libs.*` aliases.

```bash
./gradlew assembleDebug                       # debug APK (per-ABI splits + universal)
./gradlew installDebug                        # build + install on a connected device
./gradlew :app:testDebugUnitTest              # JVM unit tests (fast, no device)
./gradlew :app:connectedDebugAndroidTest      # instrumented tests (needs a device)
./gradlew :app:assembleRelease                # runs R8; the only way to catch a missing keep rule
./gradlew lint
```

Two modules: `:app` and `:tdlib` (Telegram native wrapper — see below).

### Required secrets

The Telegram feature needs an API id/hash from <https://my.telegram.org/apps>. They are obfuscated
at native-compile time. Put them in `local.properties` (or supply them as env vars in CI):

- `ID_SECRET`, `MASK_SECRET`, `HASH_SECRET` — read by `tdlib/build.gradle.kts` `getSecret()` and
  passed as CMake/cpp flags to `tdlib/src/main/cpp/native-lib.cpp`, which reconstructs the values
  at runtime and exposes them through `org.drinkless.tdlib.Secrets`. Never hardcode them in source.

`local.properties` also holds `sdk.dir` and is git-ignored. IDE tip: set
`idea.max.intellisense.filesize=5000` in `idea.properties` — `TdApi.java` is ~4.8 MB.

## Architecture

Clean-ish layering inside a single module, with Hilt DI throughout. Package root
`dev.re7gog.b_sideloader`. **Dependencies point inwards**: `ui` -> `domain` <- `data`. The domain
layer has no Android, Room, Retrofit or TDLib imports; check that before adding one.

```
core/       coroutines/  DispatcherProvider, cancellation-safe runCatching
            log/         Logger seam (debug-only chatter compiled behind a lambda)

domain/     model/       TrackedApp, AppSource, UpdateCandidate, InstallProgress, AppSettings...
            error/       AppError - the closed hierarchy every failure maps to
            repository/  AppsRepository, GithubRepository, TelegramRepository, Settings/Secrets
            installer/   InstallerGateway, PackageInspector, ApkStagingArea
            background/  BackgroundWorkScheduler, BackgroundRestrictions, BackgroundHealth
            device/      DeviceInfo
            selection/   NameMatcher, AbiMatcher, Github/TelegramApkSelector  (pure, unit-tested)
            usecase/     ObserveTrackedApps, ResolveUpdate, InstallApp, RunUpdateSweep, ...

data/       local/       Room database, DAO, entities  (+ exported schemas in app/schemas)
            remote/      Retrofit GithubApi, DTOs, mappers, OkHttp interceptors
            telegram/    TdlibClient (JNI -> coroutines) + TelegramRepositoryImpl + mappers
            installer/   session/ and privileged/ backends, event bus, staging, gateway
            background/  WorkManager scheduler, worker, monitor service, notifications, OEM quirks
            settings/    DataStore-backed settings
            encrypt/     Keystore AES-GCM + SecureSecretsRepository
            mapper/      entity <-> domain
            error/       Throwable -> AppError
            device/      AndroidDeviceInfo
            di/          Hilt modules (+ src/debug for debug-only bindings)

ui/         BSideLoaderApp.kt      navigation-suite shell + Nav3 entryProvider
            navigation/            NavKeys, NavigationState, Navigator
            common/                component/ text/ error/ permission/ util/  (shared widgets)
            feature/<name>/        Screen + ViewModel + UiState per feature
            theme/
```

### Rules that keep the layering honest

1. **The domain owns the models.** Room entities, DTOs and `TdApi` types never leave `data`; each
   has a mapper in `data/mapper` or `data/*/mapper`. UI-shaped state lives with its feature.
2. **Failures are values of one type.** Data-layer code translates its exceptions to `AppError`
   (`data/error/ThrowableToAppError.kt`, `apiCall { }`); the UI turns an `AppError` into text in
   `ui/common/error/AppErrorText.kt`, which is exhaustive — add a case there when you add one to
   `AppError`.
3. **Cancellation is never swallowed.** Use `runCatchingCancellable` / `suspendRunCatching` from
   `core/coroutines`, or rethrow via `Throwable.rethrowIfCancellation()`. A bare
   `catch (e: Exception)` around suspending code is a bug.
4. **No `Dispatchers.X` outside `DefaultDispatcherProvider`.** Inject `DispatcherProvider`.
5. **No `Context` in a ViewModel.** Produce `UiText` and resolve it in the composable.
6. **UI state is immutable.** `@Immutable data class` + `ImmutableList` (kotlinx-collections-
   immutable) so Compose can skip recomposition.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [re7gog/B-SideLoader](https://github.com/re7gog/B-SideLoader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
