---
trigger: always_on
description: Working notes for AI agents and new contributors on the droidcon Kenya Android app.
---

# AGENTS.md

Working notes for AI agents and new contributors on the droidcon Kenya Android app.

`CONTRIBUTING.md` covers the human process — issues, forks, PR expectations. This file
covers what you need to know to change code here without breaking it.

---

## Commands

Run these before opening a PR. CI runs the same set.

```bash
./gradlew spotlessApply ktlintFormat          # format first — the checks below are strict
./gradlew spotlessCheck ktlintCheck detekt    # static analysis
./gradlew lint                                # Android Lint + Slack's Compose rules
./gradlew stabilityCheck                      # Compose recomposition regressions
./gradlew assembleDebug                       # build
./gradlew testDebugUnitTest                   # JVM + Robolectric tests
```

Instrumentation tests run on Gradle Managed Devices, so no emulator setup is needed:

```bash
./gradlew :data:supportedApiLevelsGroupDebugAndroidTest    # api30 + api34
```

Single test class:

```bash
./gradlew :presentation:testDebugUnitTest --tests "*SessionsFilterStateTest*"
```

No JDK setup needed — `gradle/gradle-daemon-jvm.properties` pins the daemon to Java 17 and the
foojay resolver provisions it.

---

## Module layout

```
app                  Application, MainActivity host, manifest, DI graph root
presentation         All Compose UI, ViewModels, navigation
chai                 Design system: colours, typography, shared components
domain               Pure Kotlin — models and repository interfaces. No Android dependency.
data                 Repository implementations, sync, mappers
datasource:local     Room database, DAOs, entities
datasource:remote    Ktor client, DTOs, Remote Config
build-logic          Convention plugins
```

**Dependency rule:** `domain` depends on nothing. `data` depends on `domain`, never the
reverse. `presentation` does not reach into `datasource:local`. Keep `domain` free of
Android imports — it is the module that would move first if this ever becomes
multiplatform.

Add a module by applying the convention plugins, not by copying a `build.gradle.kts`:

```kotlin
plugins {
    alias(libs.plugins.droidconke.android.library)
    alias(libs.plugins.droidconke.android.hilt)
}
```

---

## Stack

Kotlin 2.4, AGP 9.3 on Gradle 9.7, Compose (BOM 2026.08.00, Material 3), **Navigation 3**,
Hilt + KSP, Ktor 3, Room 2.8, WorkManager, Firebase (Crashlytics, Remote Config, Messaging,
Perf). `compileSdk`/`targetSdk` 37, `minSdk` 26.

**The build uses AGP's built-in Kotlin.** `org.jetbrains.kotlin.android` is not applied
anywhere, and both `android.newDsl` and `android.builtInKotlin` are left at their AGP 9
defaults. Do not add the Kotlin Android plugin back: under the new DSL, applying it
alongside AGP's own Kotlin support is a hard error, not a warning.

Consequences worth knowing before you edit a build file:

- Library modules have no `defaultConfig.targetSdk`. Only the test APK does, via
  `testOptions.targetSdk`.
- The `android { }` block resolves to `com.android.build.api.dsl.*`, not the legacy
  `com.android.build.gradle.*` types.
- Source sets belong to AGP, so a `languageSettings` opt-in no longer reaches the compile
  tasks. Use `kotlin { compilerOptions { optIn.add(...) } }` — see `presentation`.

Navigation 3 is not Navigation 2 with a new name. Destinations are `@Serializable` keys
implementing `NavKey`; there is no `NavHost` or route strings. See
`presentation/.../common/navigation/`.

---

## Conventions

- **Formatting is enforced.** ktlint, spotless, and detekt all fail the build. Run
  `spotlessApply ktlintFormat` before committing.
- **Apache licence header** on every file. Spotless adds it.
- **detekt forbids `TODO` in comments.** Write the note as a plain sentence, or file an
  issue.
- **No baselines.** There is no lint baseline and the ktlint one is empty. A suppression goes
  in `config/lint/lint.xml` with the reason next to it, where a reviewer will see it. Rules at
  `error` are clean and must stay clean; rules at `warning` are being burned down, and
  promoting one to `error` is the last commit of the work that clears it. Counts are in
  [`docs/static-analysis.md`](docs/static-analysis.md).
- **Strings live in `strings.xml`.** No user-visible text in Kotlin.
- **Colours come from the theme**, never from the raw palette. Read
  `MaterialTheme.chaiColorsPalette` (semantic) or `MaterialTheme.colorScheme` (Material
  roles). Do not import `ChaiBlue` and friends outside `chai/colors`.
- **Lazy lists need a stable `key`.** Without one, scroll position jumps after a sync
  reorders the list.
- **ViewModels own state; composables derive it.** Do not mirror ViewModel state in a
  `remember` — that is how the UI and the data end up disagreeing after rotation.
- Test naming: backtick-quoted sentences, e.g.
  ``fun `room filter matches a real venue room name`()``.

---

## Gotchas

Each of these has already cost a bug. They are easy to reintroduce.

**`minSdk` is 26, and that is what makes `java.time` safe.** It was 24, where `java.time`
(API 26+) is absent and the app threw `NoClassDefFoundError` during the first sync unless
core library desugaring backported it. Raising the floor removed that class of crash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [droidconKE/droidconKeKotlin](https://github.com/droidconKE/droidconKeKotlin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
