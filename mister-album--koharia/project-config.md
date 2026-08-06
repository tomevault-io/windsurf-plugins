---
trigger: always_on
description: Koharia is a Komga-focused Android reader forked from Mihon `0.19.9`. It uses Kotlin/JVM 17, Jetpack Compose + Material3, Voyager, SQLDelight, Injekt, and moko-resources. The namespace remains `eu.kanade.tachiyomi`; `applicationId` is `app.koharia`.
---

# Koharia - AI Agent Guide

Koharia is a Komga-focused Android reader forked from Mihon `0.19.9`. It uses Kotlin/JVM 17, Jetpack Compose + Material3, Voyager, SQLDelight, Injekt, and moko-resources. The namespace remains `eu.kanade.tachiyomi`; `applicationId` is `app.koharia`.

## Mandatory Rules

### Preserve User Work And Secrets

- Preserve unrelated user changes; keep edits narrowly scoped.
- Do not use destructive Git commands, force-push, commit, or push unless explicitly requested.
- Treat `local.properties`, `keystore.properties`, `*.jks`, API keys, and tokens as secrets. Never print, log, or commit them.
- Confirm the destination before pushing: `github` targets GitHub, while `origin` targets the self-hosted repository.
- Pull request titles and descriptions must be written bilingually in Chinese and English.
- When debugging an emulator or physical device, prefer ADB commands. Do not use the `computer-use` skill unless the user explicitly requests device operation for validation or debugging; otherwise, provide manual validation and debugging instructions.

### Formatting And Verification

Use Windows commands:

```powershell
.\gradlew.bat spotlessApply
.\gradlew.bat spotlessCheck
.\gradlew.bat :app:compileDebugKotlin
```

- Run `spotlessApply` after Kotlin/XML edits when practical.
- Run `spotlessCheck` for formatting-sensitive work and at least `:app:compileDebugKotlin` for code, schema, or resource changes unless the user requests a lighter workflow.
- For release validation use `.\gradlew.bat :app:assembleRelease`.
- Focus checks when appropriate: `:data:generateDebugDatabaseInterface`, `:domain:test`, or relevant module tests.
- If Gradle state is stale, use `.\gradlew.bat --stop` before broader cleanup.

### Internationalization

- Shared strings live in `i18n/src/commonMain/moko-resources/base/strings.xml` and use `tachiyomi.i18n.MR`.
- Chinese-facing changes may update `zh-rCN` with `base`; avoid mass-editing other locales.
- Do not hardcode user-facing strings in composables when an `MR.strings.*` resource fits.
- Compile after i18n edits to regenerate moko-resources accessors.

### GitHub Release Notes

- Release bodies are the source for localized in-app update notes: Chinese app locales use the Chinese block, while every other locale uses English.
- Write both blocks with the exact markers below. Do not place `---` between them; put checksums or other non-changelog content after the end marker.

```markdown
<!-- koharia-release-notes:zh -->
中文更新说明
<!-- koharia-release-notes:en -->
English release notes
<!-- koharia-release-notes:end -->
```

## Project Map

| Path | Purpose |
|------|---------|
| `app/src/main/java/koharia/epub/` | Native EPUB/Readium reader, cache, pagination, settings, progress |
| `app/src/main/java/koharia/source/komga/` | Built-in Komga source, server profiles, scoped configuration |
| `app/src/main/java/koharia/komga/` | Komga API, repository, downloads, library UI |
| `app/src/main/java/eu/kanade/tachiyomi/ui/reader/` | Comic pager/webtoon reader |
| `app/src/main/java/eu/kanade/tachiyomi/data/download/` | Download and page-cache pipeline |
| `app/src/main/java/eu/kanade/tachiyomi/data/track/komga/` | Komga comic progress/history sync |
| `domain/`, `data/` | Domain contracts/interactors and SQLDelight implementations |
| `source-api/`, `source-local/` | Source ABI and local-source implementation |
| `core/`, `presentation-core/`, `presentation-widget/` | Shared utilities, archive support, UI, widgets |

Dependency versions are defined in `gradle/libs.versions.toml`; SDK, NDK, and Java versions are defined in `gradle/koharia.versions.toml`.

## Architecture

### Dependency Injection And UI

- Koharia uses Injekt, not Hilt.
- Registrations live in `AppModule.kt`, `PreferenceModule.kt`, and `DomainModule.kt`.
- Resolve with `Injekt.get<T>()` or `injectLazy<T>()`.
- Follow existing Voyager `Screen` and `StateScreenModel` patterns; use `screenModelScope`, `launchIO`, and `withIOContext` for asynchronous work.

### Database And Preferences

- SQLDelight schema, views, and migrations live under `data/src/main/sqldelight/tachiyomi/`.
- Every schema change needs a migration. Inspect existing numeric `.sqm` files and use the next consecutive number; never trust a number copied from documentation.
- Keep adapters, generated query mappers, and repository signatures aligned.
- App/preference migrations live under `app/src/main/java/koharia/core/migration/migrations/`.
- Most user configuration uses `ScopedPreferenceStore` per Komga server. Truly global settings must explicitly use the unscoped `PreferenceStore`.

## Koharia-Specific Boundaries

- Koharia is Komga-first; do not restore removed extension/browse ecosystems unless requested.
- `KomgaSource` is the only built-in network source registered by `AndroidSourceManager`.
- Confirm the launch path before reader changes: EPUB/Readium is under `koharia/epub`; comic paging/webtoon is under `ui/reader`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mister-album/Koharia](https://github.com/Mister-album/Koharia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
