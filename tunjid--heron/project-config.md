---
trigger: always_on
description: Copyright 2024 Adetunji Dahunsi
---

<!--
    Copyright 2024 Adetunji Dahunsi

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

        http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
-->

# AGENTS.md

Operational guide for coding agents working in Heron. The [README](README.md) is the deep,
human-facing reference for architecture and build/publishing; this file is the terse map of *where
things live, the pattern to copy, the commands to run, and the rules to respect*. When something here
needs more depth, it links into the README rather than restating it.

## What this is

Heron is an adaptive, reactive, offline-first [Bluesky](https://bsky.app)/ATProto client built with
Kotlin Multiplatform and Jetpack Compose. It targets **Android, iOS, and Desktop (JVM)** — there is
**no web/wasm target**. See [README · Architecture](README.md#architecture) for the full picture.

## Module map

Five kinds of modules, in a layered dependency graph: `data → ui → ui-scaffold → feature → composeApp`
(arrows point from a layer to what's built on top of it; compile-time deps run the opposite way).
The DI graph mirrors these layers — see [README · Dependency Injection](README.md#dependency-injection).

| Kind | Modules | Role |
|---|---|---|
| `data:*` | `data/core`, `data/models`, `data/database`, `data/lexicons` (generated), … | Root layer. Repositories, Room, Ktor/[Ozone](https://github.com/christiandeange/ozone) ATProto, `WriteQueue`. Depends on no other layer. Reads never error; writes are queued. |
| `ui:*` | `ui/core`, `ui/media`, `ui/timeline`, `ui/tiling`, `ui/profile` | Reusable Compose components/effects. `ui/timeline` holds `TimelineState`/`timelineStateHolder` (paging via [Tiler](https://github.com/tunjid/Tiler)). |
| `ui/scaffold` | `ui/scaffold`, `ui/sheets` | `ui/scaffold` (formerly `scaffold`) holds `AppState`, navigation, pane coordination, back previews, and exposes `PaneScaffoldState` to features. `ui/sheets` sits just above it with the app's bottom sheets, which depend on `ui/scaffold` so they self-navigate. |
| `feature:*` | ~23 destinations: `feature/feed`, `feature/profile`, `feature/home`, … | Navigation destinations (screens). `feature/template` is the shared abstraction they depend on — **not** a copy-me scaffold. |
| app modules | `composeApp`, `androidApp`, `desktopApp` | `composeApp` = shared assembly + per-platform `EntryPoint*.kt`. `androidApp`/`desktopApp` are OS launchers; iOS launches from the `iosApp` Xcode project. |

## Anatomy of a feature

Every `feature:*` module has the same four parts. `feature/feed` is the reference example
(`feature/feed/src/commonMain/kotlin/com/tunjid/heron/feed/`):

- **`State.kt`** — an `interface State` with an immutable `@Snapshottable`/`@Serializable`
  implementation, plus a sealed `Action(val key: String)` hierarchy.
- **`<Name>ViewModel.kt`** — an `@AssistedInject` ViewModel (assisted args carry the navigation
  route) that delegates to `scope.actionSuspendingStateMutator(...)`. Mutation handlers live in the
  `producer` lambda; state is produced with
  `started = SharingStarted.WhileSubscribed(FeatureWhileSubscribed)`.
- **`<Name>Screen.kt`** — a composable taking `state`, `actions: (Action) -> Unit`, and
  `PaneScaffoldState`.
- **`di/Bindings.kt`** — a `@BindingContainer` exposing two binding sets:
  - `<Name>Bindings` contributing the `PaneEntry` screen factory (`@IntoMap`) into the app's
    `entryMap`, and
  - `<Name>NavigationBindings` contributing a `RouteMatcher` (`@IntoMap`, via `urlRouteMatcher`)
    into the `routeMatcherMap`.

See [README · State production](README.md#state-production) for the Action-key parallelism model and
the Mutator pipeline.

## Recipe: add a new feature

Metro builds and verifies the DI graph at **compile time**, so a missed wiring point is a build
error with a clear message — *building the feature is the test*. The four places to touch:

1. **Create `feature/<name>/`** with a `build.gradle.kts` applying `id("kotlin-library-convention")`
   plus the Compose and serialization plugins (copy `feature/template/build.gradle.kts`), and the
   four source files above.
2. **Register the module** in [`settings.gradle.kts`](settings.gradle.kts) — add `":feature:<name>"`
   to the `include(...)` block (alongside `":feature:feed"`).
3. **Add the bindings to the graph**: `@Includes <name>Bindings` in
   [`AppGraph`](composeApp/src/commonMain/kotlin/com/tunjid/heron/di/AppGraph.kt) and
   `@Includes <name>NavigationBindings` in
   [`AppNavigationGraph`](composeApp/src/commonMain/kotlin/com/tunjid/heron/di/AppNavigationGraph.kt).
4. **Instantiate both** in the graph-factory call in
   [`EntryPoint.kt`](composeApp/src/commonMain/kotlin/com/tunjid/heron/EntryPoint.kt) (the
   `<name>Bindings = <Name>Bindings(...)` list).

## Build, test & format

Use the Gradle wrapper (`./gradlew`). Dependencies are versioned in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tunjid/heron](https://github.com/tunjid/heron) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
