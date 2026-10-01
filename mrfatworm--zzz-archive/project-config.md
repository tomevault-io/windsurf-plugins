---
trigger: always_on
description: generates per machine in `.idea/runConfigurations/`, one per Xcode configuration.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
./gradlew :desktopApp:run                    # Run desktop app
./gradlew :desktopApp:hotRun                 # Run desktop with hot reload
./gradlew lintKotlin                         # ktlint check (CI gate)
./gradlew formatKotlin                       # ktlint auto-fix
./gradlew :composeApp:testAndroidHostTest    # Unit tests (CI gate) — covers commonTest too
./gradlew :composeApp:desktopTest            # Same commonTest, run on the desktop JVM target
./gradlew :composeApp:iosSimulatorArm64Test  # Same commonTest, run on the iOS simulator (macOS only)
```

Run a single test class or method with the standard Gradle filter:

```bash
./gradlew :composeApp:testAndroidHostTest --tests "feature.agent.presentation.AgentsListViewModelTest"
./gradlew :composeApp:desktopTest --tests "*AgentsListUseCaseTest.getFactionsList*"
```

The same commands are committed as shared IDE run configurations under `.run/` — `AndroidApp Dev`,
`AndroidApp Live Install`, `DesktopApp Dev` / `Live` / `Hot Reload`, `Lint Kotlin`, `Format Kotlin`,
`Unit Tests` and `Unit Tests All Platforms`. They sit outside `.idea/` (which is gitignored) so every
machine gets the same list. iOS is not among them: it runs from the `iosApp *` entries Android Studio
generates per machine in `.idea/runConfigurations/`, one per Xcode configuration.

### Build variant

`Dev` / `Live` is decided by **one** Gradle property, `zzz.variant`, defaulting to `Dev` in
`gradle.properties`. The root `build.gradle.kts` validates it (an unknown value fails the build) and
hands it to every module through `extra["zzzVariant"]`, which `composeApp` turns into
`ZzzConfig`, `androidApp` into its product flavor, and `desktopApp` into its package id.

```bash
./gradlew :androidApp:assembleLiveRelease -Pzzz.variant=Live   # Live
./gradlew :desktopApp:run                                      # Dev, from gradle.properties
```

To build Live from the IDE, where `-P` is not available, set `zzz.variant=Live` in
`~/.gradle/gradle.properties` — it outranks the project file in Gradle's property precedence, so it
switches the machine without touching a tracked file. Never commit a non-`Dev` value to
`gradle.properties`; CI asserts the committed default, because a `Live` default would silently point
every local build at the production asset branch.

Only the flavor named by the property is created, so `assembleLiveRelease` exists **only** under
`-Pzzz.variant=Live` — a mismatched pair fails as an unknown task rather than quietly shipping the
wrong asset branch. iOS has no Gradle entry point of its own: each Xcode configuration sets a
`VARIANT` build setting (`Dev Debug` → `Dev`, `Production Release` → `Live`) and the framework build
phase forwards it as `-Pzzz.variant`.

## Architecture

### Module topology

Three Gradle modules, but only one holds code: **`:composeApp`** is the KMP module containing every
feature, the design system, DI, networking, and persistence. `:androidApp`, `:desktopApp` and
`iosApp/` are thin platform shells (entry point, platform DI bindings, packaging config) that depend
on it. Adding a feature means adding a package under `composeApp/src/commonMain/kotlin/feature/`,
not a new Gradle module.

### Feature layering

Each `feature/<name>/` is self-contained and internally layered:

```
feature/<name>/
├── presentation/   *Screen.kt, *Content, *ViewModel.kt, *Action.kt
├── domain/         *UseCase.kt
├── data/           repository/, database/, mapper/
├── model/          *Response.kt (DTO), *State.kt (UI state), domain models
└── components/     feature-local Composables
```

Cross-feature code lives at the top level: `network/`, `database/`, `datastore/`, `di/`, `ui/`,
`utils/`, `root/`.

### MVI, and where navigation actions are handled

A screen is split into a **stateful `*Screen`** (resolves the ViewModel via `koinViewModel()`,
collects `uiState`) and a **stateless `*Content`** (takes `uiState` + `onAction`). The non-obvious
part is that `*Screen` **intercepts navigation actions before they reach the ViewModel** and routes
them to nav lambdas instead:

```kotlin
onAction = { action ->
    when (action) {
        is AgentsListAction.ClickAgent -> onAgentClick(action.agentId)
        AgentsListAction.ClickBack -> onBackClick()
        else -> viewModel.onAction(action)
    }
}
```

So navigation-flavoured branches inside a ViewModel's `onAction` are unreachable — they exist only
to keep the `when` exhaustive. ViewModels never touch the NavController.

State is a single `data class *State` in `model/`, held in a `MutableStateFlow`, exposed via
`stateIn(viewModelScope, SharingStarted.WhileSubscribed(15000L), …)` with initial work kicked off in
`.onStart { }`.

### Data flow: Room is the source of truth

Repositories do **not** return network responses to the UI. The pattern is:

1. `requestAndUpdate<X>DB()` fetches from Ktor and writes entities into Room, returning `Result<Unit>`.
2. `get<X>()` returns a `Flow` straight from the DAO, mapped entity → domain model.
3. The UI observes that Flow; a refresh is a write to Room, which re-emits.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mrfatworm/ZZZ-Archive](https://github.com/mrfatworm/ZZZ-Archive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
