---
trigger: always_on
description: **Calculator Plus** is a single-module Android calculator app (package `com.gigaworks.tech.calculator`) supporting both a standard numpad and a scientific pad. Current release is **v3.0.0** (versionCode 75, targetSdk 36, minSdk 23).
---

# Calculator Plus — Agent Guidance

## Project Overview

**Calculator Plus** is a single-module Android calculator app (package `com.gigaworks.tech.calculator`) supporting both a standard numpad and a scientific pad. Current release is **v3.0.0** (versionCode 75, targetSdk 36, minSdk 23).

---

## Build System

| Tool | Version |
|---|---|
| Gradle | 9.4.1 |
| Kotlin | 2.3.21 |
| KSP | 2.3.8 |
| AGP | 9.2.0 |
| Java compatibility | VERSION_17 |

### Key Gradle Commands

```bash
./gradlew assembleDebug          # debug APK
./gradlew bundleRelease          # release AAB (for Play Store)
./gradlew test                   # run unit tests
./gradlew lint                   # lint check
./gradlew bumpVersionCode        # increment VERSION_CODE in version.properties
./gradlew printVersionName       # prints current versionName
./gradlew clean                  # clean build directory
```

### Version Management

- `versionCode` lives in `version.properties` as `VERSION_CODE` — **never edit this manually**, the `bumpVersionCode` Gradle task owns it.
- `versionName` is hardcoded in `app/build.gradle` `defaultConfig.versionName` — update this manually before a release.
- Release signing secrets (`SIGNING_KEY`, `ALIAS`, `KEY_STORE_PASSWORD`, `KEY_PASSWORD`) are GitHub Actions secrets — do not hardcode them.

---

## Architecture

Clean layered architecture within a single `app` module:

```
ui/           → Activities, Fragments, ViewModels, custom Views
├── base/     → BaseActivity<B: ViewBinding>, BaseFragment<B: ViewBinding>
├── main/     → MainActivity, MainViewModel, helper functions, CalculatorPadViewPager
├── history/  → HistoryActivity, HistoryViewModel, HistoryAdapter
├── settings/ → SettingsActivity, SettingsViewModel, SettingsHelper
└── about/    → AboutActivity, fragments for About/Changelog/OpenSource

domain/       → Pure data classes: History, HistoryAdapterItem, License
repository/   → HistoryRepository (suspending functions, wraps Room DAO)
cache/        → Room database, HistoryDao, HistoryEntity, CacheUtil, Response<T>
di/           → Hilt modules: AppModule (AppPreference), CacheModule (Room DB)
util/         → AppPreference, constants, extension functions, enums, Logger
```

**Dependency injection:** Hilt (`@AndroidEntryPoint`, `@HiltViewModel`, `@Inject`). All ViewModels use `@HiltViewModel`.

**View binding:** Enabled project-wide. Every activity/fragment uses `getViewBinding(inflater)` via the generic `BaseActivity<B>` / `BaseFragment<B>` pattern. Never use `findViewById`.

**Navigation:** Activity-based (no single-activity nav graph). `AboutActivity` uses Jetpack Navigation internally for its three fragments.

---

## Key Files

| File | Purpose |
|---|---|
| `app/build.gradle` | App-level dependencies, build types, SDK versions |
| `build.gradle` | Root classpath, Kotlin/Hilt/Firebase plugin versions |
| `version.properties` | `VERSION_CODE` — auto-managed by CI |
| `app/src/main/res/xml/remote_config_defaults.xml` | Default values for Firebase Remote Config keys |
| `app/src/main/java/.../util/AppPreference.kt` | All SharedPreferences keys in one place |
| `app/src/main/java/.../util/Constants.kt` | Firebase Analytics event name constants |
| `app/src/main/java/.../ui/main/helper/Evaluate.kt` | Core expression evaluator using `ch.obermuhlner:big-math` |
| `app/src/main/java/.../ui/main/helper/HandleButtonClicks.kt` | Pure functions for expression string manipulation |
| `app/src/main/java/.../ui/main/helper/NumberFormat.kt` | Number separator, rounding, formatting helpers |
| `app/src/main/java/.../util/GoogleMobileAdsConsentManager.kt` | GDPR/UMP consent flow wrapper |

---

## Calculation Engine

The evaluator lives in `ui/main/helper/Evaluate.kt`. Key facts:

- Uses `ch.obermuhlner:big-math:2.3.2` for arbitrary-precision arithmetic — never swap this for `java.lang.Math`.
- Throws `CalculationException(msg: CalculationMessage)` for four error states: `INVALID_EXPRESSION`, `DIVIDE_BY_ZERO`, `VALUE_TOO_LARGE`, `DOMAIN_ERROR`.
- `prepareExpression(expr)` normalises the string before evaluation (converts display symbols to math operators).
- `tryBalancingBrackets(expr)` is called by `MainViewModel` when the expression is unbalanced and Smart Calculation is on.
- Trig functions respect the `AngleType` (DEG/RAD) preference passed into `getResult(expression, angleType)`.

**Never call the evaluator on the main thread for long expressions.** `MainViewModel.calculateExpression()` runs synchronously but is called from `afterTextChanged`; keep expression prep fast.

---

## Preferences (AppPreference)

All preference keys are `const val` in `AppPreference.companion`. The SharedPreferences file name equals `BuildConfig.APPLICATION_ID`. Known keys:

| Key constant | Default | Purpose |
|---|---|---|
| `ANGLE_TYPE` | `DEG` | Trig angle unit |
| `NUMBER_SEPARATOR` | `INTERNATIONAL` | Thousands separator style |
| `SMART_CALCULATION` | `true` | Auto-balance brackets |
| `ANSWER_PRECISION` | `6` | Decimal places in result |
| `MEMORY` | `""` | Stored memory value |
| `EXPRESSION` | `""` | Saved expression across sessions |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arch10/Calculator-Plus](https://github.com/arch10/Calculator-Plus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
