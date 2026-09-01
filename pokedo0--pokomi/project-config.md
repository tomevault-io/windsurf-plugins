---
trigger: always_on
description: Pokomi is an Android manga reader (min SDK 26, target SDK 36, JVM 17 / Kotlin) forked from **Mihon** + **TachiyomiSY**. Stack: Jetpack Compose + Material3, Voyager navigation, SQLDelight, Injekt DI. `applicationId`: `app.pokomi`.
---

# Pokomi – AI Agent Guide

Pokomi is an Android manga reader (min SDK 26, target SDK 36, JVM 17 / Kotlin) forked from **Mihon** + **TachiyomiSY**. Stack: Jetpack Compose + Material3, Voyager navigation, SQLDelight, Injekt DI. `applicationId`: `app.pokomi`.

---

## Mandatory rules for AI agents

**Read this section before every change.** These rules override shortcuts (e.g. copying nearby `MR` imports or only running `compileDebugKotlin`).

### Git

| Rule | Required behavior |
|------|-------------------|
| Branch | Create a **feature branch** for the task (`git checkout -b <type>/<short-description>`). |
| Commit | **OK** on a feature branch when work is ready. **Never** commit directly to `master` / `main` unless the user explicitly asks. |
| Push | **OK** to push the **current feature branch** when work is ready. **Never** push to `master` / `main` unless the user explicitly asks. |

Before `git push`, confirm the current branch is not `master` or `main` (`git branch --show-current`).

### Internationalization (strings)

| String kind | Module | Resource class | Base folder only |
|-------------|--------|----------------|------------------|
| Pokomi-only / current fork features (new UI, Following, author grouping, PKM blocks, post-rename app behavior) | `i18n-pkm/` | **`PKMR`** | `i18n-pkm/src/commonMain/moko-resources/base/` |
| Komikku-only (new features, KMK UI, library-update errors, WebDAV, Discord, etc.) | `i18n-kmk/` | **`KMR`** | `i18n-kmk/src/commonMain/moko-resources/base/` |
| Shared Mihon / upstream behavior | `i18n/` | **`MR`** | `i18n/src/commonMain/moko-resources/base/` |
| TachiyomiSY-only | `i18n-sy/` | **`SYMR`** | `i18n-sy/src/commonMain/moko-resources/base/` |

**Hard rules:**

- **Never** add Pokomi/current-fork strings to `i18n-kmk/`, `i18n/`, or `i18n-sy/`.
- **Never** add Komikku-specific strings to `i18n/` or `i18n-sy/`.
- **Never** edit non-`base` locale `strings.xml` or `plurals.xml` files in `i18n-kmk/`, `i18n/`, or `i18n-sy/` (Weblate owns translations).
- For `i18n-pkm/`, add new strings to `base/` and also add Simplified/Traditional Chinese translations in `zh-rCN/` and `zh-rTW/` when the string is user-facing.
- Import: `import tachiyomi.i18n.pkm.PKMR` for Pokomi/current-fork strings.
- Import: `import tachiyomi.i18n.kmk.KMR` for Komikku strings.
- If a change is inside `// PKM -->` … `// PKM <--` or adds Pokomi/current-fork behavior, default to **`PKMR` + `i18n-pkm`**.
- If a change is inside `// KMK -->` … `// KMK <--` or adds Komikku-only behavior, default to **`KMR` + `i18n-kmk`**.

### Formatting & build verification

**“Build passes” is not enough.** After Kotlin/XML edits, run **in this order** before marking work complete:

```bash
./gradlew spotlessApply    # fix formatting
./gradlew spotlessCheck    # must pass (same as CI)
./gradlew assembleDebug    # or :app:compileDebugKotlin for a faster compile-only check
```

- **Do not** skip `spotlessCheck` when verifying changes.
- If `spotlessCheck` fails, run `spotlessApply` and re-run `spotlessCheck`.
- On Cloud VM, export `ANDROID_HOME` and `JAVA_HOME` first (see [Cursor Cloud](#cursor-cloud-specific-instructions)).

---

## Module layout

| Module | Purpose |
|--------|---------|
| `app/` | UI (`eu.kanade.*`, `exh/`, `mihon/`), DI, workers, build variants |
| `domain/` | Use cases in `…/interactor/` (e.g. `GetManga`), models, repo interfaces |
| `data/` | SQLDelight DB, `*RepositoryImpl` (`tachiyomi.data.*`) |
| `core:common/` | Network (OkHttp), security, storage, shared utils |
| `core:archive/` | CBZ/archive reading with optional encryption |
| `core-metadata/` | Comic-info metadata parsing |
| `source-api/` / `source-local/` | Extension `Source` API + local source |
| `presentation-core/` | Shared Compose components |
| `presentation-widget/` | Home-screen Glance widget |
| `i18n/` | Mihon strings → `MR` (moko-resources) |
| `i18n-kmk/` | Komikku strings → `KMR` |
| `i18n-pkm/` | Pokomi/current-fork strings → `PKMR` |
| `i18n-sy/` | TachiyomiSY strings → `SYMR` |
| `flagkit/` | Country-flag drawables |
| `telemetry/` | Firebase/Crashlytics (noop unless `-Pinclude-telemetry`) |
| `macrobenchmark/` | Macrobenchmark tests |

Dependency flow: `app` → `domain` → `source-api`; `data` implements `domain` repos.

Version catalogs: `gradle/libs.versions.toml`, `kotlinx.versions.toml`, `androidx.versions.toml`, `compose.versions.toml`, `sy.versions.toml`.

---

## Architecture

**DI** – `uy.kohesive.injekt` (not Hilt). Register in `AppModule.kt`, `DomainModule.kt`, `KMKDomainModule.kt`, `SYDomainModule.kt` via `addSingleton` / `addSingletonFactory`. Resolve with `Injekt.get<T>()` or `injectLazy<T>()`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pokedo0/Pokomi](https://github.com/pokedo0/Pokomi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
