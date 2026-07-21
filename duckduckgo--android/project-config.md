---
trigger: always_on
description: DuckDuckGo Android is a privacy-focused browser with 100+ Gradle modules. The app provides built-in search, tracker blocking, HTTPS enforcement, and other privacy features.
---

# AGENTS.md — DuckDuckGo Android Browser

## Project Overview

DuckDuckGo Android is a privacy-focused browser with 100+ Gradle modules. The app provides built-in search, tracker blocking, HTTPS enforcement, and other privacy features.

**Versions** (SDK levels, Kotlin, Gradle, libraries) live in the build files — don't restate them here:
`min_sdk` / `target_sdk` / `compile_sdk` in `build.gradle`, `version.kotlin` in `versions.properties`,
and the Gradle version in `gradle/wrapper/gradle-wrapper.properties`.
**Build:** AGP via refreshVersions. DI is Anvil/Dagger2 today, with a migration to **Metro** in flight
(dual-build selected by the `ddg.di` Gradle property; see `build.gradle`).
**Toolchain:** Kotlin JVM target 17; building requires **JDK 21** (Metro compiler plugin).

---

## Build System

### Module Discovery

`settings.gradle` auto-discovers modules up to 2 levels deep — any directory containing `build.gradle` is included automatically. No manual `include` is needed.

## Build & Test Commands

```bash
# Unit tests (all modules)
./gradlew jvm_tests

# Unit tests for a single module
./gradlew :my-feature-impl:testDebugUnitTest

# Code quality (spotless + lint + unit tests)
./gradlew jvm_checks

# Lint only
./gradlew lint_check

# Code formatting check / fix
./gradlew spotlessCheck
./gradlew spotlessApply

# Install app
./gradlew installInternalRelease    # internal build (more testing features)
./gradlew installPlayRelease        # play store build

# Maestro UI tests (requires app installed on device)
maestro test .maestro/autofill/1_autofill_shown_in_overflow.yaml   # single test
maestro test .maestro/autofill                                      # all in directory
maestro test .maestro --include-tags releaseTest                    # by tag
```

Note: `jvm_tests` and `jvm_checks` resolve to `testPlayDebugUnitTest` in `:app` and `testDebugUnitTest` in library modules. To run a single test class, use `--tests`:
```bash
./gradlew :my-feature-impl:testDebugUnitTest --tests "com.duckduckgo.my.feature.RealFooTest"
```

### Proprietary Fonts

The app uses a proprietary DuckSans font from a private GitHub Packages repository. The build conditionally swaps between the proprietary `ddg-proprietary-fonts` AAR and a local `:fonts` fallback module (empty `<font-family/>` stubs) based on credential availability. See `build.gradle` for the credential detection logic and `android-design-system/fonts/readme.md` for details.

### Build Variants

| Dimension | Flavors |
|---|---|
| store | `internal`, `fdroid`, `play` |
| Build types | `debug`, `release`, `upload` |

---

## Module Architecture

Every feature follows an `-api` / `-impl` split:

```
my-feature/
  my-feature-api/       ← interfaces, data classes, no implementation
  my-feature-impl/      ← implementation, UI, DI bindings
```

Key rules enforced at build time (`build.gradle`):
- `-api` modules cannot use Anvil, depend on Dagger (except `:feature-toggles-api`, `:settings-api`), depend on other `-api` modules (except `:feature-toggles-api`, `:navigation-api`, `:js-messaging-api`), or depend on `:di`
- Only `:app` can depend on `-impl` modules — features communicate through `-api` only
- `-internal` modules must use `internalImplementation` configuration
- No KAPT anywhere except `:app` — use KSP
- No `strings.xml` outside `:app` — use `strings-<feature>.xml`
- Android tests only allowed in: `app`, `sync-lib`, `httpsupgrade-impl`, `pir-impl`, `feature-toggles-impl`
- No module can depend on `:app`

`settings.gradle` auto-discovers modules up to 2 levels deep — any directory with `build.gradle` is included. New `-impl` modules must be added to `app/build.gradle` to enter the Dagger graph.

## Dependency Injection (Anvil / Dagger)

Scopes: `AppScope` (app lifetime), `ActivityScope` (single activity), `FragmentScope` (viewmodels/fragments).

Use `@SingleInstanceIn(AppScope::class)` — never `@Singleton` (lint enforces this).

```kotlin
// Binding
@SingleInstanceIn(AppScope::class)
@ContributesBinding(AppScope::class)
class RealFoo @Inject constructor(...) : Foo

// ViewModel
@ContributesViewModel(FragmentScope::class)
class FooViewModel @Inject constructor(...) : ViewModel()

// Plugin (multibinding)
@ContributesMultibinding(AppScope::class)
class MyPlugin @Inject constructor() : SomePlugin

// Remote feature flag
@ContributesRemoteFeature(scope = AppScope::class, featureName = "myFeature")
interface MyFeature : Feature {
    @Toggle.DefaultValue(DefaultFeatureValue.INTERNAL)
    fun myToggle(): Toggle
}
```

Activity context: inject `@ActivityContext Context` at `ActivityScope` via DI, don't pass Context through interfaces.

App coroutine scope: `@AppCoroutineScope private val appScope: CoroutineScope`.

## Plugin System

Two kinds of plugin points:

- **`PluginPoint<T>`** (`@ContributesPluginPoint`): basic multibinding, returns all registered plugins. Use `@PriorityKey(n)` for ordering (lower = higher priority).
- **`ActivePluginPoint<T>`** (`@ContributesActivePluginPoint`): wraps plugins with remote feature flag gating. Plugins extend `ActivePlugin`, declared via a private trigger interface. Generated flags default to TRUE.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckduckgo/Android](https://github.com/duckduckgo/Android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
